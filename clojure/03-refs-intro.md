!SLIDE bullets incremental transition=fade
# Clojure reference types

* Vars: global per thread value 
* Agents: asynchronous single object store 
* Atoms: simple CAS semantics
* Refs: STM managed references 
* All implements ARef (some IFn)
* Muatated by applying fn 

!SLIDE center
# Reference categorization 

|             | Asynchronous |  Synchronous |
|:------------|:------------:|:------------:|
| Coordinated |              |     Ref      |
| Independent |   Agent      |    Atom      |


