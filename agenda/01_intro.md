!SLIDE 
# Clojure concurrency #

!SLIDE bullets 
# Agenda

* Concurrency and why it matters 
* Current approaches (and why they fail)
* Clojure concurrency take
* Clojure STM drill down 


!SLIDE bullets 
.notes 
Concurrency deals with structuring our software to handle non deterministic control flow , parallelism deals with running operation on multiple cores for better throughput (http://tinyurl.com/bv483my).
# Concurrency and why is matters?

* The end of free lunch
* Concurrency != Parallelism

!SLIDE bullets 
.notes 
Livelock Threads are often busy responding to one another, if two threads are responding endlessly to one another they are livelocked, it similar to two people passing in the same corridor each trying to give head way to another.
Starvation happens when a thread tries to access a shared resource but get little or not access to it, if for example one threads calls frequently a long synchronized method on an object it locks that object for other threads.

# Common issues

* Deadlocks
* Livelocks 
* Race conditions
* Starvation 


