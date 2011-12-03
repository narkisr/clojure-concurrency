!SLIDE bullets incremental transition=fade

# Common techniques 

* Locks
* Actors
* Dataflow
* STM

!SLIDE bullets incremental transition=fade

# Locks

* Prevalent in most languages today
* Manual low level mechanism 
* Very error prone


!SLIDE code execute

    @@@ clojure
    (def o (Object.))
    (defn sleep-lock [interval] 
      (future 
        (locking o 
         (do 
          (Thread/sleep interval) 
          (str (java.util.Date.))
        ))))

    (def f1 (sleep-lock 3000))
    (def f2 (sleep-lock 1000))
    (println (str "longer " @f1))
    (println (str "shorter " @f2))
    
!SLIDE bullets incremental transition=fade

# Lock pitfalls 

* Taking too few locks  
* Taking the wrong locks 
* Taking locks in the wrong order
* Error recovery
* Lost wake-ups and erroneous retries
   
