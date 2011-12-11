!SLIDE subsection
.notes vals include commuted values (unlike sets)
# Snapshot isolation  

!SLIDE code style small 
    @@@ java
    // modified by current TX Refs -> in-transaction values. 
    final HashMap<Ref, Object> vals = new HashMap<Ref, Object>();

    // Refs modified by current TX by using ref-set or alter 
    final HashSet<Ref> sets = new HashSet<Ref>(); 
   
!SLIDE code smaller
.notes Here we see how ref value is obtained within the TX (alter calls it), if it doesn't have a TX value the value is looked for in the tvals
# doget

    @@@ java
    Object doGet(Ref ref){
      if(!info.running())
            throw retryex;
      if(vals.containsKey(ref))
            return vals.get(ref);
      try
            {
            ref.lock.readLock().lock();
            if(ref.tvals == null)
                   throw new IllegalStateException(ref.toString() + " is unbound.");
            Ref.TVal ver = ref.tvals;
            do
                  {
                    if(ver.point <= readPoint)  {
                      return ver.val;
                    }
                  } while((ver = ver.prior) != ref.tvals);
           }
      finally
            {
            ref.lock.readLock().unlock();
            }
      //no version of val precedes the read point
      ref.faults.incrementAndGet();
      throw retryex;
     }

!SLIDE code smaller
# doset 

    @@@ java
    Object doSet(Ref ref, Object val){
      if(!info.running())
             throw retryex;
      if(commutes.containsKey(ref))
            throw new IllegalStateException("Can't set after commute");
      if(!sets.contains(ref))
            {
            sets.add(ref);
            lock(ref);
            }
      vals.put(ref, val);
      return val;
    }


