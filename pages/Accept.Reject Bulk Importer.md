title:: Accept/Reject Bulk Importer

- Metrics tagging
	- ```ruby
	  statsd.increment('pipeline_stage.failure', tags: ['stage:{STAGE_CLASS_NAME}'])
	  '])
	  ```
	- Looking at this again, it'll probably be easier to drop the idea of tags and just use the dot notation for the pipeline stages
- Should the need some way of following up on transfers that were accepted with warnings, we can give ops another view which shows every transfer_out_import where the `transfer_state` is "accepted" but the `import_warnings` field is not empty.