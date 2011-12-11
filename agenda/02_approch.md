!SLIDE bullets 

# Common techniques 

* Locks
* Actors
* Dataflow
* STM

!SLIDE bullets 
# Locks

* Prevalent in most languages today
* Manual low level mechanism 
* Very error prone

!SLIDE code execute
.notes A basic lock example, notice that the shorter version  cannot complete until the longer one releases its lock

    @@@ clojure
    (def o (Object.))
    (defn sleep-lock [interval] 
      (future 
        (locking o 
         (do 
          (Thread/sleep interval) 
          (str (java.util.Date.))
        ))))

    (def f1 (sleep-lock 1000))
    (def f2 (sleep-lock 500))
    (println (str "longer " @f1))
    (println (str "shorter " @f2))
    
!SLIDE bullets
 
# Lock pitfalls 

* Taking too few locks  
* Taking the wrong locks 
* Taking locks in the wrong order
* Error recovery
* Hard to compose
   
