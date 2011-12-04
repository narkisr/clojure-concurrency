!SLIDE bullets incremental transition=fade
# Clojure reference types

* Vars: global per thread value 
* Atoms: simple CAS semantics
* Refs: STM managed references 
* Agents: asynchronous single object store

!SLIDE center
# Reference categorization 

|             | Asynchronous |  Synchronous |
|:------------|:------------:|:------------:|
| Coordinated |              |     Ref      |
| Independent |   Agent      |    Atom      |

!SLIDE code execute
# Vars

    @@@ clojure
    (def v 1)

    (alter-var-root #'*out* (constantly *out*))

    (defn set-5 [o] 
      (binding [v o] ; v bound to TL
        (set! v 5)
        (println "p1:" v))
      )

    (let [thread (Thread. #(set-5 v))]
      (.start thread)
      (.join thread)
      (println "p2:" v)) 


!SLIDE code execute
# Agents

    @@@ clojure
    (def a (agent 5)) 
 
    (dotimes [i 5] 
      (send a #(do (Thread/sleep 100) (inc %))))

    (println "p1:" @a)

    (await a)
    
    (println "p2:" @a)
