!SLIDE bullets incremental transition=fade
# Refs

* Coordinate storage using STM
* Mutation: reset!, swap!, ref-set, alter, commute 

!SLIDE bullets incremental transition=fade
.notes  Defines a unit of work, commit if successful, retry if conflict (up to RETRY_LIMIT currently 10,000)
# Transactions 

![Persistent vector](transaction.svg "transaction")

!SLIDE center

<embed id="embed"  type="image/svg+xml" src="image/clojure/collision.svg"/>



