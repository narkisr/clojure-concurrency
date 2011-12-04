!SLIDE bullets incremental transition=fade
# Clojure reference types

* Vars: global per thread value 
* Atoms: simple CAS semantics
* Refs: STM managed references 
* Agents: asynchronous single object store

!SLIDE center
# Reference categorization 

|             | Asynchronous |  Synchronous |
|:------------|:------------:|:------------:|
| Coordinated |              |     Ref      |
| Independent |   Agent      |    Atom      |


