!SLIDE bullets incremental transition=fade
# Atoms 

* Synchronous non transactional storage
* Has CAS semantics
* Mutation: reset!, compare-and-set! and swap!

!SLIDE code execute
.notes replaces the current value without taking older value into account
# reset!

    @@@ clojure
    (def x (atom 1))
    (reset! x 2)
    (println @x)

!SLIDE code execute
.notes CAS semantics
# compare-and-set!

    @@@ clojure
    (def x (atom 1))

    (alter-var-root #'*out* (constantly *out*))

    (defn update-atom []
      (let [curr-val @x]
      (println "update-atom: curr-val =" curr-val) ; -> 1
      (Thread/sleep 50) ; give reset! time to run
      (println
        (compare-and-set! x curr-val (inc curr-val))))) ; -> false

    (let [thread (Thread. #(update-atom))]
      (.start thread)
	(Thread/sleep 25) ; give thread time to call update-atom
      (reset! x 3) ; happens after update-atom binds curr-val
      (.join thread)) ; wait for thread to finish

      (println @x) ; -> 3

!SLIDE code execute
.notes swap! repeatedly replay the function until not collision is made
# swap! 

    @@@ clojure
    (alter-var-root #'*out* (constantly *out*))

    (def x (atom 1))


    (defn update-atom [curr-val]
      (println "update-atom: curr-val =" curr-val)
      (Thread/sleep 50) ; give reset! time to run
      (inc curr-val))

    (let [thread (Thread. #(swap! x update-atom))]
      (.start thread)
      (Thread/sleep 25) ; give swap! time to call update-atom
      (reset! x 3)
      (.join thread)) ; wait for thread to finish

    (println @x) ; -> 4

!SLIDE bullets incremental transition=fade
# Usage

* Global state without coordination (memoization)
* Reducing STM overhead
