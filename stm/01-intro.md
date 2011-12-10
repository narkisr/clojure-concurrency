!SLIDE bullets incremental transition=fade
# Clojure STM
.notes 
MVCC uses increasing transaction IDs, it ensures a transaction never has to wait for an object by maintaining several versions of an object.
Transaction executing under snapshot isolation appears to operate on a personal snapshot [of the database], taken at the start of the transaction. When the transaction concludes, it will successfully commit only if the values updated by the transaction have not been changed externally since the snapshot was taken. 
       

* Implementation based upon MVCC 
* Each Ref maintains multiple versions (tvals)
* Transaction operate under snpashot isolation


!SLIDE code small
.notes LockingTransaction is the object which runs the dosync actions, it gets the thread local LockingTransaction (if exists)
# dosync

    @@@ java
    public class LockingTransaction{ 

     final static ThreadLocal<LockingTransaction> transaction = new ThreadLocal<LockingTransaction>(); 

     static public Object runInTransaction(Callable fn) throws Exception{
       LockingTransaction t = transaction.get();
       if(t == null)
             transaction.set(t = new LockingTransaction());
 
       if(t.info != null)
            return fn.call();
 
       return t.run(fn);
     } 
    
!SLIDE code small
# run 

    @@@ java
    code
