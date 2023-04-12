- #+BEGIN_QUERY
  {:title "All page tags"
  :query [:find ?tag-name
        :where
        [?tag :page/name ?tag-name]]
  :view (fn [tags]
        [:div
         (for [tag (flatten tags)]
           [:a.tag.mr-1 {:href (str "/page/" tag)}
            (str "#" tag)])])}
  #+END_QUERY