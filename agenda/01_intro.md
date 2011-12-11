!SLIDE 
# Clojure concurrency #

!SLIDE bullets 
# Agenda

* Concurrency and why it matters 
* Current approaches (and why they fail)
* Clojure concurrency take


!SLIDE bullets 
.notes Setting up some context
# Concurrency and why is matters?

* The end of free lunch
* Concurrency != Parallelism

!SLIDE bullets 
.notes livelock occurs when concurrently running threads are performing work (as opposed to be being blocked, waiting on resources), but cannot complete due to something that other threads have done or not done.  
# Common issues

* Deadlocks
* Livelocks 
* Race conditions
* Starvation 


