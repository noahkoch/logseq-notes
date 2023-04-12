alias:: templates

- Usage:
	- Create new node
	- Right click on bullet
	- Convert to template
	- Add properties
	- Use `/template` to implement
- Get tagged items:
  ```
  #+BEGIN_QUERY
  {:title "Therapy Sessions"
   :query [:find (pull ?b [*])
         :where
         [?p :block/name "tag-name"]
         [?b :block/ref-pages ?p]]}
  #+END_QUERY
  ```
- template:: Book Note
  tags:: books
  book::
  note:: ""
- template:: I Want This
  tags:: [[finance]], [[wish list]], [[things]] 
  item::
  cost::
  why::
  when::
- template:: Quebec Restaurant
  tags:: [[Quebec Restaurants]], [[Restaurants]], [[Quebec City]] 
  city:: Quebec
  name::
  type::
  notes::
- {{query(property template) }}
  query-table:: false