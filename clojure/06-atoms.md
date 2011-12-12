!SLIDE bullets incremental 
# Atoms 

* Synchronous non transactional storage
* Has CAS semantics
* Mutation: reset!, compare-and-set! and swap!

!SLIDE code execute
.notes replaces the current value without taking older value into account

    @@@ clojure
    (def x (atom 1))
    (reset! x 2)
    (println @x)

!SLIDE code execute
.notes This compare-and-set! fails since we reset curr-val before the second thread had a chance to CAS.

    @@@ clojure
    (def x (atom 1))

    (alter-var-root #'*out* (constantly *out*))

    (defn update-atom []
      (let [curr-val @x]
        (println "update-atom: curr-val =" curr-val) ; -> 1
        (Thread/sleep 50) ; give reset! time to run
        (println
          (compare-and-set! x curr-val (inc curr-val))))) ; -> false

    (let [thread (Thread. update-atom)]
      (.start thread)
      (Thread/sleep 25) 
      (reset! x 3) ; taken after curr-val set
      (.join thread)) 

      (println @x)

!SLIDE code execute
.notes unlike CAS which fails, swap! repeatedly replay the function until no collision is made we can see that its running twice

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
      (.join thread)) 

    (println @x)

!SLIDE bullets incremental 
# Usage

* Global state without coordination (memoization)
* Reducing STM overhead

!SLIDE code 
.notes deref state and ctor, here we see that the atom stores the reference in an atomic container

    @@@ java
    final public class Atom extends ARef{
    final AtomicReference state;

    public Atom(Object state){
      this.state = new AtomicReference(state);
    }

    ...

    public Object deref(){
	return state.get();
    } 
  
    
!SLIDE code 
notes. reset!, compare-and-set implementation, nothing too exciting 

    @@@ java
     public Object reset(Object newval){
       Object oldval = state.get();
       validate(newval);
       state.set(newval);
       notifyWatches(oldval, newval);
       return newval;
    }  
    
    public boolean compareAndSet(Object oldv, Object newv){
      validate(newv);
      boolean ret = state.compareAndSet(oldv, newv);
      if(ret)
          notifyWatches(oldv, newv);
      return ret;
    } 

!SLIDE code 
.notes swap! implementation, we see the loop as long as the compareAndSet fail the loop will carry on

    @@@ java
    public Object swap(IFn f, Object x, Object y, ISeq args) {
      for(; ;)
          {
            Object v = deref();
            Object newv = f.applyTo(RT.listStar(v, x, y, args));
            validate(newv);
            if(state.compareAndSet(v, newv))
                  {
                  notifyWatches(v, newv);
                  return newv;
                  }
          }
    }  
