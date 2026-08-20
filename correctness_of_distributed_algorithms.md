# Correctness of Distributed Algorithms

distributed systems are not just concurrent but have potential for partial
failure and don't have access to the same clock.

TLA book <https://lamport.azurewebsites.net/tla/science.pdf>:
  * precedence: $\square F \vee G = (\square F) \vee G$
  * weak fairness means roughly that a process cannot forever be able to do a state
      transition but not do any

proofs of correctness of distributed algorithms:
* see material on proving safety and liveness in TLA book
* Raft: thesis ch 8
* paxos: Ghosh - Distributed Systems (§13.4.1)

TCP is reliable under temporary link failures, see
Lynch - Distributed Algorithms (§ 22.5.3)

paxos or the randomized consensus alg in Lynch - Distributed Algorithms (21.3)
don't explicitly track which peers have failed, they just wait for e.g.
$n - f$ responses so they're ok if $f$ agents fail. however, Paxos doesn't
really have a liveness guarantee.
on the other hand, raft does use a failure detector and has stronger liveness
https://cs.stackexchange.com/questions/102860/does-the-paxos-algorithm-use-failure-detectors
I'm surprised other algorithms that use a failure-detection oracle like
Ghosh - Distributed Systems (§13.5.1.2) aren't used in practice

probabilistic failure detectors Hayashibara - The/spl Phi/accrual Failure Detector
* used in Cassandra


## tools

testing
* https://github.com/jepsen-io/jepsen

formal specifications
* TLA+
  * Raft specified
  * Paxos specified and proof checked
  * TLA+ book https://lamport.azurewebsites.net/tla/book-21-07-04.pdf
  * in TLA+ you're verifying a specification against its specification
  * cheatsheet: https://mbt.informal.systems/docs/tla_basics_tutorials/tla+cheatsheet.html
  * incorporating z3: https://mbt.informal.systems/docs/tla_basics_tutorials/apalache_vs_tlc.html
  * the model checker can test safety as well as liveness properties
  * if a fairness condition is not part of the description of the algorithm, processes can stutter forever. it's
    probably more convenient to explicitly have a `hasCrashed` variable for each process, and
    [include fairness](https://www.binwang.me/2020-10-06-Understand-Liveness-and-Fairness-in-TLA.html)
* others
  * Isabelle https://lawrencecpaulson.github.io/2022/10/12/verifying-distributed-systems-isabelle.html
  * https://spinroot.com/spin/whatispin.html
  * process calculus https://pron.github.io/posts/tlaplus_part3
