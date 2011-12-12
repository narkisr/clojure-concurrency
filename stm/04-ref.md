!SLIDE subsection
# Ref

!SLIDE code small
.notes Each Ref stores multiple versions of committed values in a circular list tvals
# Ref

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


!SLIDE bullets incremental 
.notes Adding the faulted value into the tvals list reduces the likelihood for other TX's to have faults (assuming they started after the committing TX)
# Fault

* A TX read a ref that has no local value
* All existing tvals were committed after TX has began
* This causes a retry and increment the faults [counter](/#54)
* First commit on faulty Ref will be added to tvals 

!SLIDE bullets incremental 
.notes A knows that according to the tinfo field on the Ref
# Write conflict and Barging

* TX A try modifying a Ref that TX B changed without committing
* A will try to Barge B

!SLIDE code small
.notes 
In order for A to barge B (else A will retry):
1. A must have been running for at least 1/100th of a second (BARGE_WAIT_NANOS)
2. A started before B (favors older txns)
3. B has a status of RUNNING and can be changed to KILLED
If B status is set to KILLED it will retry

    @@@ java
    private boolean barge(Info refinfo){
      boolean barged = false;
      // if this transaction is older
      // try to abort the other
      if(bargeTimeElapsed() && startPoint < refinfo.startPoint)
            {
        barged = refinfo.status.compareAndSet(RUNNING, KILLED);
        if(barged)
            refinfo.latch.countDown();
            }
      return barged;
    }  


