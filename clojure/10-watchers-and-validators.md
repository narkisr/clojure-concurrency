!SLIDE code execute 
.notes We can set validators on var/ref/agent/atom 

    @@@ clojure
    (alter-var-root #'*out* (constantly *out*))

    (def num (ref 0 :validator integer?))

    (try
      (dosync
        (ref-set num 1) 
        ; no commit, transaction is aborted
        (ref-set num "foo")) 
          (catch Exception e (println (str e))))


!SLIDE code execute 
.notes We can add watchers, the watch fn will be called synchronously, on the agent's thread if an agent, before any pending sends if agent or ref. 

    @@@ clojure
    (alter-var-root #'*out* (constantly *out*))

    (def num (ref 0))

    (add-watch num nil 
      (fn [key ref old new] (println old new )))

      (dosync
        (ref-set num "foo"))


    
