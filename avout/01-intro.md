!SLIDE subsection
# Distributed STM


!SLIDE bullets incremental 
# Avout

* Uses zookeeper to manage distributed locks
* Has Refs for different backends like MongoDB, Zookeeper and local-ref
* Similar mutation semantics reset!!, swap!!, dosync!!, ref-set!!, alter!!, commute!! 

!SLIDE code execute

    @@@ clojure 
    (use 'avout.core)
    (def client (connect "127.0.0.1"))
 
    (def r0 (zk-ref client "/r0" 0))
    (def r1 (zk-ref client "/r1" []))
 
    (dosync!! client
      (alter!! r0 inc)
      (alter!! r1 conj @r0))
 
    (println @r1)
