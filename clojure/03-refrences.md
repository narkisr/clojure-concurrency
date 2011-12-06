!SLIDE bullets incremental transition=fade
# Clojure reference types

* Vars: global per thread value 
* Agents: asynchronous single object store 
* Atoms: simple CAS semantics
* Refs: STM managed references 
* All implements ARef (some IFn)

!SLIDE center
# Reference categorization 

|             | Asynchronous |  Synchronous |
|:------------|:------------:|:------------:|
| Coordinated |              |     Ref      |
| Independent |   Agent      |    Atom      |

!SLIDE bullets incremental transition=fade
# Vars intro

* Per thread global store
* defn and def use var
* Dynamic bindings 
* Mutation: set!, def, defn

!SLIDE code execute
# Vars example

    @@@ clojure
    (def v 1)

    (alter-var-root #'*out* (constantly *out*))

    (defn set-5 [o] 
      (binding [v o] ; v bound to TL
        (set! v 5)
        (println "p1:" v))
      )

    (let [thread (Thread. #(set-5 v))]
      (.start thread)
      (.join thread)
      (println "p2:" v)) 

!SLIDE bullets incremental transition=fade
# Vars usage 

* Per thread storage is required
* Configuration (like \*out\*)
* AOP in functions


!SLIDE code 
# Implementation ctor

    @@@ java
    Var(Namespace ns, Symbol sym, Object root){
      this(ns, sym);
      this.root = root;
      ++rev;
    }

!SLIDE code small
.notes 
 2. dvals holds current Frame which holds bindings and previous Frame, 
 3. pushThreadBindings invoked by bindings, stores a new Frame into dvals,
 4. A symmetric popThreadBindings pulls previous Frame.
# Framing implementation 

    @@@ java

    public final class Var extends ARef implements IFn, IRef, Settable{

      // ... 

      static final ThreadLocal<Frame> dvals = new ThreadLocal<Frame>(){
        // ...
      };
      
      // ... 
   
      public static void pushThreadBindings(Associative bindings){
        Frame f = dvals.get();
        Associative bmap = f.bindings;
        for(ISeq bs = bindings.seq(); bs != null; bs = bs.next())
              {
              IMapEntry e = (IMapEntry) bs.first();
              Var v = (Var) e.key();
  
               if(!v.dynamic)
                     throw new IllegalStateException(String.format("Can't dynamically bind non-dynamic var: %s/%s", v.ns, v.sym));

              v.validate(v.getValidator(), e.val());
              v.threadBound.set(true);
              bmap = bmap.assoc(v, new TBox(Thread.currentThread(), e.val()));
              }
        dvals.set(new Frame(bmap, f));
      }
   
      // ...
    }

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

!SLIDE bullets incremental transition=fade
# Agents Usage
 
* Accessing single threaded components
* Side effects in STM transactions
* Queue processing 

!SLIDE code small
# send implementation 

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
    public class Agent extends ARef { // ...

      AtomicReference<ActionQueue> aq = new AtomicReference<ActionQueue>(ActionQueue.EMPTY);

      public Object dispatch(IFn fn, ISeq args, boolean solo) {
        // ... 
        Action action = new Action(this, fn, args, solo);
        dispatchAction(action);

        return this;
      } // ...  

!SLIDE code small 
.notes dispatchAction gets the current transaction, if it exists it sets the action on it
       enqueue runs in a tight loop, only one Thread will be able to escape it at a time:
       1. If  A is on aq.get() and B is on aq.compareAndSet then A will get B.aq,
       2.  once B is set it goes out with prior that has 0 elements and 
# dispatch implementation cont .. 

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
     } // ... 

     void enqueue(Action action){
       boolean queued = false;
       ActionQueue prior = null;
       while(!queued)// if we didn't manage to CAS 
             {
             prior = aq.get();
             queued = aq.compareAndSet(prior, new ActionQueue((IPersistentStack)prior.q.cons(action), prior.error));
             }
 
       if(prior.q.count() == 0 && prior.error == null) // this is always guaranteed since only one thread will escape the while loop at a time
             action.execute();// pop is done within the action see while(!popped)
     } // ... 

    } // closing class
    

!SLIDE bullets incremental transition=fade
# Atoms intro

* points

!SLIDE code execute
# Atoms 

    @@@ clojure
    code

!SLIDE bullets incremental transition=fade
# Atoms usage

* points
