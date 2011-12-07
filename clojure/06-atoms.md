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

!SLIDE code 
# deref state and ctor

    @@@ java
    final public class Atom extends ARef{
    final AtomicReference state;

    public Atom(Object state){
      this.state = new AtomicReference(state);
    }

    public Atom(Object state, IPersistentMap meta){
      super(meta);
      this.state = new AtomicReference(state);
    }

    public Object deref(){
	return state.get();
    } 
  
    
!SLIDE code 
# reset!  compare-and-set

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
# swap! 

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
