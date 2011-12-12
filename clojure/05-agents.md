!SLIDE bullets incremental 
# Agents intro

* Single value store
* Manipulated via messages (Actor like)
* Mutation: send, send-of


!SLIDE code execute
.notes This example show how we send tasks to the agent and wait for it to finish processing, we use send-off and not send since it blocks, the difference is with the thread pools that will be used to process this, see next..

    @@@ clojure
    (def a (agent 5)) 
 
    (dotimes [i 5] 
      ; we use send-of and not send
      (send-off a #(do (Thread/sleep 100) (inc %))))

    (println "p1:" @a)

    (await a)
    
    (println "p2:" @a)

!SLIDE bullets incremental 
# Agents Usage
 
* Accessing single threaded components
* Side effects in STM transactions
* Queue processing 

!SLIDE code small
.noted The send implementation is calling dispatch

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

!SLIDE code smaller
.notes dispatch creates a new actions and dispatches it

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

!SLIDE code smaller
.notes dispatchAction gets the current transaction, if it exists it sets the action on it
       The Agent's queue implementation is based on Treiber's algorithm (see http://tinyurl.com/37mydc) for non blocking stack.

    @@@ java
     static void dispatchAction(Action action){
       LockingTransaction trans = LockingTransaction.getRunning();
       if(trans != null)// if called within transaction
            trans.enqueue(action);
       else if(nested.get() != null)
             {
             nested.set(nested.get().cons(action));
             }
      else
         action.agent.enqueue(action);
     } 

     ... 

     void enqueue(Action action){// push
       boolean queued = false;
       ActionQueue prior = null;
       while(!queued)// while we didn't manage to CAS 
             {
             prior = aq.get();
             queued = aq.compareAndSet(prior, 
                new ActionQueue((IPersistentStack)prior.q.cons(action), prior.error));
             }
 
       if(prior.q.count() == 0 && prior.error == null)// only first action 
             action.execute();
     } 


!SLIDE code smaller
.notes The execute invokes the action, if send is used pooledExecutor is used (closer to the cpu count of threads less context switching) if send-of is used then a cached thread pool is used, a new Thread (potentially) will be created for each action (good for blocking actions).

      @@@ java
	void execute(){
		try
			{
			if(solo)
				soloExecutor.execute(this);// dynamic pool 
			else
				pooledExecutor.execute(this);// fixed size pool (cpu cores + 2)
			}
		catch(Throwable error)
			{
			if(agent.errorHandler != null)
				{
				try
					{
					agent.errorHandler.invoke(agent, error);
					}
				catch(Throwable e) {} // ignore errorHandler errors
				}
			}
	}
    
!SLIDE code smaller
.notes pop implementation, each one of these is running a seperate thread, the if before the finally is pulling the next message
 
    @@@ java
     static void doRun(Action action){// invoked from run in a Thread 
            try
                  {
                  nested.set(PersistentVector.EMPTY);

                  Throwable error = null;
                  try
                      {// the current message processing
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
