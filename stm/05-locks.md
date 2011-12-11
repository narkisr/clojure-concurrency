!SLIDE bullets 
.notes 
1. We saw that locks are held briefly in the doSet doGet methods, 
2. The one case that a look is held for the entire TX is if ensure was called
3. The tinfo field save us the need to lock the Ref for the entier TX since we can check for pending writers 
# locks

* Each ref has ReentrantReadWriteLock
* Many TX's can read only one can write
* Lock free strategy
  * Lock are held briefly (not entire TX)* 
  * Ref.tinfo marks an active write

!SLIDE code smaller
.notes Here we see the locking method which is called from doSet, here we can see how write conflict is detected using the tinfo field on the Ref

    @@@ java

    Object lock(Ref ref){
      releaseIfEnsured(ref); //can't upgrade readLock, so release it

      boolean unlocked = true;
      try
            {
            tryWriteLock(ref);// trying to get the write lock to the ref 
            unlocked = false;

            if(ref.tvals != null && ref.tvals.point > readPoint)
                   throw retryex;
            Info refinfo = ref.tinfo;

            //write lock conflict
            if(refinfo != null && refinfo != info && refinfo.running()) // did any TX has uncommited write?
                  {
                  if(!barge(refinfo))
                        {
                        ref.lock.writeLock().unlock();
                        unlocked = true;
                        return blockAndBail(refinfo);
                        }
                  }
            ref.tinfo = info;
            return ref.tvals == null ? null : ref.tvals.val;
            }
      finally
            {
            if(!unlocked)
                  ref.lock.writeLock().unlock();
            }
    }

