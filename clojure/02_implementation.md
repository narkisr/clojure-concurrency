!SLIDE subsection

# Implementation 

!SLIDE bullets incremental transition=fade

# Persistent data structures

* Implemented as trees
* Immutable
* Data sharing 

!SLIDE code style execute
# Immutability 

    @@@ clojure
    (def a [1 2 3])
    
    (conj a 4)

    (println a)
