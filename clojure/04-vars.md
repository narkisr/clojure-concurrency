!SLIDE bullets incremental transition=fade
# Vars 

* Per thread global store
* defn and def use var
* Dynamic bindings 
* Mutation: set!, def, defn

!SLIDE code execute
# set! and binding

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
# Usage 

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


