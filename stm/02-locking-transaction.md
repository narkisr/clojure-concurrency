!SLIDE subsection
# LockingTransaction

!SLIDE code smaller
.notes The LockingTransaction is held in thread local storage, the info class holds the transaction status, most status are self explanatory KILLED is set when one TX barges another
info is also a part of lock-free strategy to mark Refs as having an uncommitted change

    @@@ java
    public class LockingTransaction{ 

      final static ThreadLocal<LockingTransaction> transaction =
                                 new ThreadLocal<LockingTransaction>(); 

      static final int RUNNING = 0; // executing code
      static final int COMMITTING = 1; // committing changes to refs
      static final int RETRY = 2; // will be retied
      static final int KILLED = 3; // barged 
      static final int COMMITTED = 4; // done committing

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
               ret = fn.call();// calling the body of dosync
               } 

            ...

           }
    }


