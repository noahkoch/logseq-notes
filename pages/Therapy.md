- Collect all the notes from therapy here
- query-table:: false
  query-properties:: [:block]
  #+BEGIN_QUERY
  {:title "Therapy Sessions"
   :query [:find (pull ?b [*])
         :where
         [?p :block/name "therapy-session"]
         [?b :block/ref-pages ?p]]}
  #+END_QUERY
- #+BEGIN_QUERY
  {:title "Things to bring up"
   :query [:find (pull ?b [*])
         :where
         [?p :block/name "therapy-concerns"]
         [?b :block/ref-pages ?p]]}
  #+END_QUERY