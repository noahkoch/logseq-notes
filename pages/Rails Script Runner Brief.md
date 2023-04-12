alias:: rtrb

- ### Why?
collapsed:: true
	- We are preparing to turn off production rcon access on all Wealthsimple services outside of declared incidents
	- Money Movement relies heavily on rcon for on call
	- Scripts will fill a gap for a few things:
		- Things that we don't do often enough to warrant the development time needed to build a tool
		- Tasks that will only be done once
			- e.g. backfills
	- Scripts should not be used for:
		- Things we do regularly where the time to develop a tool is relatively insignificant
		- Urgent matters which are deserving of a tool but the task needs to be unblocked now
			- An incident should be declared for this instead
	- Why use scripts instead of rake tasks?
		- Rake tasks were originally considered
		- Rake tasks are designed to be run from the CLI and thus have too much weight for what we need
		- It is technically feasible to execute a rake task from a controller or background job, but, it essentially loads up an entire rails environment (slow) and the code is really ugly
	- When should we just build a tool?
		- Each _execution_ of a script should be assumed to be 30 minutes of developer time
			- This includes the actual execution of the task
			- The time spent context switching
			- The time spent remembering/finding the URL for the script runner
			- The time spent parsing DD for success/failure messages
		- We would assume that a tool (in Atlas, for example) would be able to be run by a non-developer, those a tool's dev time would only be in the creation of the tool
		- Based on those assumptions, we could use the formula for the true developer cost of using the script runner
			- `(time_to_create_script + (30 minutes * executions_in_the_next_12_months)) < time_to_develop_tool`
			- In cases where the formula comes out to a wash/pretty close, we should default to building a tool
			- Example Scenario:
				- A script takes **4 hours** to create (code, testing and deployment), a tool takes **100 hours** (planning, frontend code, backend code, design, testing, deployment)
				- If we forecast the tool being executed **once per week** over the next year:
					- 4 hours + (30 minutes * 52) = 30 hours
					- This means a script would be a fine choice since it would take fewer developer hours over the next
				- If we forecast the tool being executed **three times per week** over the next year:
					- 4 hours + (30 minutes * 156) = 90 hours
					- 90 and 100 are pretty close to one another, we should opt for the tool
				- If we forecast the tool being executed **every weekday** over the next year:
					- 4 hours + (30 minutes * 365) = Way more than 100 hours
					- A tool is a no-brainer in this situation
- ### How?
	- **Basic Rails RESTful controller**
		- GET - `index`
			- Show a list of whitelisted rake script
				- Anything in the `lib/scripts` folder should be considered whitelisted
			- Shows a list of UUIDs from recent script runs
		- GET - `show`
			- Display the input options for a particular rake script and a "run" button
			- If no input options, just show the run button
		- POST - `run`
			- Non-default rails controller method
			- Steps
				- Generate a UUID
				- Log that we queued a job to run this script
				- Store the UUID in a cookie along with a timestamp
					- This is how we'll show the recent UUIDs on the index page
					- People are likely to execute the script and then forget to copy down the flash message before closing or refreshing the page
					- In the future we could create a DB, but that's more than what's needed here
				- Trigger job to run the script with the options and UUID
				- Return the UUID on execution
	- **Script Runner Job**
		- Log script started
		- Increment metric associated with this script
		- Executes the job if the required params are provided
		- Adds a log to DD with who executed the script, which script and with what options
			- Consider PII here
		- Each script execution would have a UUID (generated in the controller) associated to make it easy to find in Datadog
			- Standard info and error logs should be created in the parent class for use in the child class which automatically tags the UUID
		- Log script finished/errored
	- **Script Configuration**
		- The script itself should always have a standard skeleton which gives us some basic protections. But, this should also be simple enough to not get in the way of the task at hand. We want this to be run in a variety of one off solutions, this may include something high priority (just shy of an incident).
		- **Proposed solution**
		  
		  ```ruby
		  # The actual script class, very lightweight, script writer doesn't need to worry about
		  # any validation handling beyond the `param` definition on the first line.
		  class ChildKlass < ParentKlass
		    params required: [:param1], optional: [:param2, :param3]
		  
		    def perform(params)
		      # We're doing a thing!
		    end
		  end
		  ```
		  ```ruby
		  # The parent class that does the pesky validation and whatnot
		  class ParentKlass
		    @required_params = []
		    @optional_params = []
		  
		    def self.run(params)
		      # validations on the params
		      # remove any params that weren't permitted (optionally or required)
		      new.perform(params)
		    end
		  
		    def self.required_params
		      @required_params
		    end
		  
		    def self.optional_params
		      @optional_params
		    end
		  
		    def self.params(options)
		      @required_params = options[:required]
		      @optional_params = options[:optional]
		    end
		  end
		  ```
	- **Structure**
		- Controller to be placed in a `tools` namespace in `app/controllers`
		- Routes placed in a `tools` namespace
			- Limited to `index`, `show` and `run`
		- Scripts placed in `lib/tasks`
			- This follows the lead of how rails structures rake tasks
- ### Other Considerations
collapsed:: true
	- From the proposed structure of the script runner, there would be no way to view output
		- Datadog logging should be sufficient, the UUID generated by the controller will make this output easy to track down
		- If we're trying to pull specific information using the script runner, we'll run into limitations with using DD since we can't send DD PII
	- In the script runner job, we'll be keeping track of how many times a script gets run
		- A datadog monitor on this metric could trip if we go over X usages in Y time to make sure we're not using the script runner as a replacement for good tooling