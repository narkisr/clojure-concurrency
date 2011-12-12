!SLIDE bullets incremental transition=fade
# Agents intro

* Single value store
* Manipulated via messages (Actor like)
* Mutation: send, send-of


!SLIDE code execute
.notes an example on how await works
# Agents example

    @@@ clojure
    (def a (agent 5)) 
 
    (dotimes [i 5] 
      (send a #(do (Thread/sleep 100) (inc %))))

    (println "p1:" @a)

    (await a)
    
    (println "p2:" @a)

!SLIDE bullets incremental 
# Agents Usage
 
* Accessing single threaded components
* Side effects in STM transactions
* Queue processing 

!SLIDE code small
.noted send implementation is calling dispatch

    @@@ clojure
    (defn send
     "Dispatch an action to an agent. Returns the agent immediately.
      Subsequently, in a thread from a thread pool, the state of the agent
      will be set to the value of:

     (apply action-fn state-of-agent args)"
     {:added "1.0"
      :static true}
      [^clojure.lang.Agent a f & args]
      (.dispatch a (binding [*agent* a] (binding-conveyor-fn f)) args false)) 

!SLIDE code small 
.notes dispatch creates a new actions and dispatches it
# dispatch implementation

    @@@ java
    public class Agent extends ARef { 
     
      ...

      AtomicReference<ActionQueue> aq = new AtomicReference<ActionQueue>(ActionQueue.EMPTY);

      public Object dispatch(IFn fn, ISeq args, boolean solo) {

         ... 

        Action action = new Action(this, fn, args, solo);
        dispatchAction(action);

        return this;
      } 

!SLIDE code small 
.notes dispatchAction gets the current transaction, if it exists it sets the action on it
       The Agent's queue implementation is based on Treiber's algorithm (see http://tinyurl.com/37mydc).

    @@@ java
     static void dispatchAction(Action action){
       LockingTransaction trans = LockingTransaction.getRunning();
       if(trans != null)
            trans.enqueue(action);
       else if(nested.get() != null)// not sure what nested is
             {
             nested.set(nested.get().cons(action));
             }
      else
         action.agent.enqueue(action);
     } 

     ... 

     void enqueue(Action action){ // push
       boolean queued = false;
       ActionQueue prior = null;
       while(!queued)// if we didn't manage to CAS 
             {
             prior = aq.get();
             queued = aq.compareAndSet(prior, new ActionQueue((IPersistentStack)prior.q.cons(action), prior.error));
             }
 
       if(prior.q.count() == 0 && prior.error == null) // only first action 
             action.execute();
     } 

    
!SLIDE code smaller
.notes pop implementation
 
    @@@ java
     static void doRun(Action action){// invoked from run in a Thread 
            try
                  {
                  nested.set(PersistentVector.EMPTY);

                  Throwable error = null;
                  try
                        {
                        Object oldval = action.agent.state;
                        Object newval =  action.fn.applyTo(RT.cons(action.agent.state, action.args));
                        action.agent.setState(newval);
                        action.agent.notifyWatches(oldval,newval);
                        }
                 catch(Throwable e)
                        {
                        error = e;
                        }

                  ... 

                  boolean popped = false;
                  ActionQueue next = null;
                  while(!popped)
                        {
                        ActionQueue prior = action.agent.aq.get();
                        next = new ActionQueue(prior.q.pop(), error);
                        popped = action.agent.aq.compareAndSet(prior, next);
                        }

                  if(error == null && next.q.count() > 0) // pulling more actions
                       ((Action) next.q.peek()).execute();
                  }
            finally

             ...	

       }
