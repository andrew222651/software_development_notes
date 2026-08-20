# Fundamental Distributed Problems

Conferences: 
* [Principles of Distributed Computing](https://dl.acm.org/conference/podc)

this field is "the union of two [...] sub-communities. One is mostly focussing
on the combined impact of asynchronism and faults [...], while the other is
mostly focussing on the impact of network structural properties [e.g. leader
election in a ring, spanning trees]"
Fraigniaud - Distributed Computational Complexities. if we don't care about the network structure we can assume it
is complete with uniform link costs.

"Synchronizers simulate an execution of a failure-free synchronous system in a
failure-free asynchronous system"

consensus seems to be "the" fault-tolerance problem. problems like leader
election and mutex assume no faults. 3 Phase Commit only solves the distributed
transaction problem for synchronous systems Guerraoui - Revisiting the Relationship Between Non-Blocking Atomic Commitment and Consensus (Sec 1)

textbooks
* the practical side: Kleppmann - Designing Data-Intensive Applications, Steen - Distributed Systems
* Ghosh - Distributed Systems (Section IV) "Faults and Fault-Tolerant Systems"
* Kshemkalyani - Distributed Computing
  * §13 Checkpointing and rollback recovery
* Lynch - Distributed Algorithms
  * § 21.3 "A Randomized Algorithm"
  * § 21.6 "Approximate Agreement"

mutual exclusion, i.e. distributed locks: according to
Steen - Distributed Systems (§5.3.6), in practice this is done via a centralized server which is equivalent
to postgres locking w/ `idle_in_transaction_session_timeout`.
However, to avoid holding a database connection, specialized services like [etcd](https://etcd.io/docs/v3.6/dev-guide/api_concurrency_reference_v3/) can be used.
Alternatively, use a database table with a unique `lock_name` column. Acquire by repeatedly connecting and attempting to insert, release by deleting.

according to Kleppmann - Designing Data-Intensive Applications (p336-...),
CAP theorem should really be "either consistency (i.e. consistency model
stronger than eventual consistency) or availability under network failure",
since if the replicas can't communicate you can either serve requests without
synchronization or not serve requests at all. e.g. cockroachdb [chooses
consistency](https://www.cockroachlabs.com/blog/limits-of-the-cap-theorem/)
instead of availability

## reductions

x -> y means you can use x to solve y

Omega aka eventually strong = failure detector s.t.
* Every failed node is eventually detected as failed
* Eventually, at least one correct node will not be suspected as failed by any
  node

Omega -> consensus
* if majority of processes are correct
* Chandra - The Weakest Failure Detector for Solving Consensus

a different failure detector -> consensus
* in any environment
* Delporte-Gallet - The Weakest Failure Detectors to Solve Certain Fundamental Problems in Distributed Computing

a complicated failure detector -> distributed transaction
* Delporte-Gallet - The Weakest Failure Detectors to Solve Certain Fundamental Problems in Distributed Computing

eventually perfect failure detector =
* Every failed node is eventually detected as failed
* Eventually, no correct node is detected as failed

eventually perfect failure detector -> leader election
* Pick highest ID correct node as leader
* https://ucbrise.github.io/cs262a-spring2018/notes/09-ds-overview-consensus.pdf

total order broadcast <-> consensus
* Défago - Total Order Broadcast and Multicast Algorithms (Sec. 2.4.5)

total order broadcast <-> replicated state machine

consensus !-> distributed transaction
* Guerraoui - Revisiting the Relationship Between Non-Blocking Atomic Commitment and Consensus
