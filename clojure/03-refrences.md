!SLIDE bullets incremental transition=fade
# Clojure reference types

* Vars: global per thread value 
* Agents: asynchronous single object store 
* Atoms: simple CAS semantics
* Refs: STM managed references 

!SLIDE center
# Reference categorization 

|             | Asynchronous |  Synchronous |
|:------------|:------------:|:------------:|
| Coordinated |              |     Ref      |
| Independent |   Agent      |    Atom      |

!SLIDE bullets incremental transition=fade
# Vars intro

* points

!SLIDE code execute
# Vars example

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

!SLIDE bullets incremental transition=fade
# Vars usage 

* Per thread storage is required
* Clojure has build in Vars (*out*)
* def and defn


!SLIDE bullets incremental transition=fade
# Agents intro

* points

!SLIDE code execute
.notes an example on how await works
# Agents example

    @@@ clojure
    (def a (agent 5)) 
 
    (dotimes [i 5] 
      (send a #(do (Thread/sleep 100) (inc %))))

    (println "p1:" @a)

    (await a)
    
    (println "p2:" @a)

!SLIDE bullets incremental transition=fade
# Agents Usage
 
* Accessing single threaded components
* Side effects in STM transactions
* Queue processing 

!SLIDE bullets incremental transition=fade
# Atoms intro

* points

!SLIDE code execute
# Atoms 

    @@@ clojure
    code

!SLIDE bullets incremental transition=fade
# Atoms usage

* points
