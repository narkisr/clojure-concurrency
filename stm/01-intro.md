!SLIDE bullets incremental transition=fade
# Clojure STM
.notes 
Transaction executing under snapshot isolation appears to operate on a personal snapshot [of the database], taken at the start of the transaction. When the transaction concludes, it will successfully commit only if the values updated by the transaction have not been changed externally since the snapshot was taken. 
       

* Implementation based upon MVCC 
* Transaction operate under snpashot isolation
* Optimistic, assumes low chances of write collisions
* No deadlocks, livelocks or race conditions possible
* Implemention core: Ref and LockingTransaction 

!SLIDE code smaller
.notes 
The LockingTransaction is held in thread local storage 
The info class holds the transaction status, most status are self explanatory KILLED is set when one TX barges another
# Locking transaction

    @@@ java
    public class LockingTransaction{ 

      final static ThreadLocal<LockingTransaction> transaction =
                                           new ThreadLocal<LockingTransaction>(); 

      static final int RUNNING = 0;
      static final int COMMITTING = 1;
      static final int RETRY = 2;
      static final int KILLED = 3;
      static final int COMMITTED = 4; 

      public static class Info{
        final AtomicInteger status;
        final long startPoint;
        final CountDownLatch latch;


        public Info(int status, long startPoint){
              this.status = new AtomicInteger(status);
              this.startPoint = startPoint;
              this.latch = new CountDownLatch(1);
        }

        public boolean running(){
              int s = status.get();
              return s == RUNNING || s == COMMITTING;
        }
      }

!SLIDE code small
.notes runInTransaction is called from the dosync macro, it gets the thread local LockingTransaction (if exists)
# runInTransaction

    @@@ java
    static public Object runInTransaction(Callable fn) throws Exception{
      LockingTransaction t = transaction.get();
      if(t == null)
            transaction.set(t = new LockingTransaction());

      if(t.info != null)
           return fn.call();
 
      return t.run(fn);
    } 


!SLIDE code smaller
.notes 
The run method is the core of the transaction lifecycle, in it the retry loop is running 
# Run

    @@@ java
    Object run(Callable fn) throws Exception{
         
      ...

      for(int i = 0; !done && i < RETRY_LIMIT; i++)
           {
           try
               {
               getReadPoint();// tracks order of retries across all transactions.
               if(i == 0)
                    {
                     startPoint = readPoint;
                     startTime = System.nanoTime();
                    }
               info = new Info(RUNNING, startPoint);
               ret = fn.call();
               } 

            ...

           }
    }
    
 
    

!SLIDE code small
.notes each Ref stores multiple versions of committed values in a circular list
# Tvals

    @@@ java
    public class Ref extends ARef implements IFn, Comparable<Ref>, IRef{ 
     ...

     public static class TVal{
       Object val;// the committed value
       long point;// transaction commit id
       long msecs;// TVal creation time
       TVal prior;// older commits
       TVal next;// newer commits
     } 
   
     TVal tvals;
     ...
    }

!SLIDE center
.notes Values are added if faults happend
# Ref history list

![Tvals version](tvals-versions.svg "tvals")


!SLIDE bullets incremental transition=fade
# Fault

* A TX read a ref that has no local value
* All existing values were commited after TX has began
* This causes a retry and tvals grow

!SLIDE bullets incremental transition=fade
.notes Barging favors old transactions 
# Write conflict and Barging

* TX A try modifying a Ref that B changed without committing
* A will try to Barge B

!SLIDE code small
.notes 
A must have been running for at least 1/100th of a second (BARGE_WAIT_NANOS)
A started before B (favors older txns)
B has a status of RUNNING and can be changed to KILLED
If B isn't barged A will retry
# Barge 

    @@@ java
    private boolean barge(Info refinfo){
      boolean barged = false;
      //if this transaction is older
      //  try to abort the other
      if(bargeTimeElapsed() && startPoint < refinfo.startPoint)
            {
        barged = refinfo.status.compareAndSet(RUNNING, KILLED);
        if(barged)
            refinfo.latch.countDown();
            }
      return barged;
    }  

    Object run(Callable fn) throws Exception{ 

       // within the retry loop      
      Info refinfo = ref.tinfo; 
      if(refinfo != null && refinfo != info && refinfo.running())
           {
             if(!barge(refinfo))
                throw retryex;
           }  

    }

