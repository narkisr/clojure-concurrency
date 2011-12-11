!SLIDE bullets incremental transition=fade
# Clojure STM

* Implementation based upon MVCC 
* Transaction operate under snpashot isolation
* Optimistic, assumes low chances of write collisions
* No deadlocks, livelocks or race conditions possible
* Implemention core: Ref and LockingTransaction 


