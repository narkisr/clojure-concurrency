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
.notes Yes locks in Clojure are possible, a but surprissing but the good java interop will enable Clojure in Clojure 
A basic lock example, notice that the shorter version cannot complete until the longer one releases its lock

    @@@ clojure
    (def o (Object.))
    (defn sleep-lock [interval] 
      (future 
        (locking o 
          (do 
            (Thread/sleep interval) 
            (str (java.util.Date.))))))

    (def f1 (sleep-lock 500))
    (def f2 (sleep-lock 100))
    (println (str "longer " @f1))
    (println (str "shorter " @f2))
    
!SLIDE bullets
.notes In erroneous situation its very hard to recover and decide which lock to release take etc..
# Lock pitfalls 

* Taking too few locks  
* Taking the wrong locks 
* Taking locks in the wrong order
* Error recovery
* Hard to compose
   
