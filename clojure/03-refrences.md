!SLIDE bullets incremental transition=fade
# Clojure reference types

* Vars: global per thread value 
* Agents: asynchronous single object store 
* Atoms: simple CAS semantics
* Refs: STM managed references 

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
# Var implementation constructor

    @@@ java
    Var(Namespace ns, Symbol sym, Object root){
      this(ns, sym);
      this.root = root;
      ++rev;
    }

!SLIDE code small
# Vars implementation bindings 

    @@@ java
    static final ThreadLocal<Frame> dvals = new ThreadLocal<Frame>(){

      protected Frame initialValue(){
           return new Frame();
      }
    };

    public static void pushThreadBindings(Associative bindings){
      Frame f = dvals.get();
      Associative bmap = f.bindings;
      for(ISeq bs = bindings.seq(); bs != null; bs = bs.next())
            {
            IMapEntry e = (IMapEntry) bs.first();
            Var v = (Var) e.key();

        if(v.sym!=null && v.sym.name.equals("hell-yeah!")){
            System.out.println();
        }
            if(!v.dynamic)
                   throw new IllegalStateException(String.format("Can't dynamically bind non-dynamic var: %s/%s", v.ns, v.sym));
            v.validate(v.getValidator(), e.val());
            v.threadBound.set(true);
            bmap = bmap.assoc(v, new TBox(Thread.currentThread(), e.val()));
            }
      dvals.set(new Frame(bmap, f));
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
