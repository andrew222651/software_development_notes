# Fundamental Distributed Problems

Conferences: 
* [Principles of Distributed Computing](https://dl.acm.org/conference/podc)

this field is "the union of two [...] sub-communities. One is mostly focussing
on the combined impact of asynchronism and faults [...], while the other is
mostly focussing on the impact of network structural properties [e.g. leader
election in a ring, spanning trees]"
<abbr title="Fraigniaud - Distributed computational complexities: are you volvo-addicted or nascar-obsessed? (2010)" data-bibtex="@inproceedings{dist2cultures,
    author = &quot;Fraigniaud, Pierre&quot;,
    title = &quot;Distributed computational complexities: are you volvo-addicted or nascar-obsessed?&quot;,
    year = &quot;2010&quot;,
    isbn = &quot;9781605588889&quot;,
    publisher = &quot;Association for Computing Machinery&quot;,
    address = &quot;New York, NY, USA&quot;,
    url = &quot;https://doi.org/10.1145/1835698.1835700&quot;,
    doi = &quot;10.1145/1835698.1835700&quot;,
    booktitle = &quot;Proceedings of the 29th ACM SIGACT-SIGOPS Symposium on Principles of Distributed Computing&quot;,
    pages = &quot;171–172&quot;,
    numpages = &quot;2&quot;,
    keywords = &quot;mobile computing, network computing, wait-free computing&quot;,
    location = &quot;Zurich, Switzerland&quot;,
    series = &quot;PODC &#x27;10&quot;
}
">[F10]</abbr>. if we don't care about the network structure we can assume it
is complete with uniform link costs.

"Synchronizers simulate an execution of a failure-free synchronous system in a
failure-free asynchronous system"

consensus seems to be "the" fault-tolerance problem. problems like leader
election and mutex assume no faults. 3 Phase Commit only solves the distributed
transaction problem for synchronous systems <abbr title="Guerraoui - Revisiting the relationship between non-blocking atomic commitment and consensus (1995)" data-bibtex="@InProceedings{guerraoui,
    author = &quot;Guerraoui, Rachid&quot;,
    editor = &quot;H{\&#x27;e}lary, Jean-Michel and Raynal, Michel&quot;,
    title = &quot;Revisiting the relationship between non-blocking atomic commitment and consensus&quot;,
    booktitle = &quot;Distributed Algorithms&quot;,
    year = &quot;1995&quot;,
    publisher = &quot;Springer Berlin Heidelberg&quot;,
    address = &quot;Berlin, Heidelberg&quot;,
    pages = &quot;87--100&quot;,
    isbn = &quot;978-3-540-44783-2&quot;
}
">[G95, Sec 1]</abbr>

textbooks
* the practical side: <abbr title="Kleppmann - Designing data-intensive applications: The big ideas behind reliable, scalable, and maintainable systems (2017)" data-bibtex="@book{kleppmann2017designing,
    author = &quot;Kleppmann, Martin&quot;,
    title = &quot;Designing data-intensive applications: The big ideas behind reliable, scalable, and maintainable systems&quot;,
    year = &quot;2017&quot;,
    publisher = {&quot; O&#x27;Reilly Media, Inc.&quot;}
}
">[K17]</abbr>,<abbr title="Steen and Tanenbaum - Distributed systems (2017)" data-bibtex="@book{van2017distributed,
    author = &quot;Van Steen, Maarten and Tanenbaum, Andrew S&quot;,
    title = &quot;Distributed systems&quot;,
    year = &quot;2017&quot;,
    publisher = &quot;Maarten van Steen Leiden, The Netherlands&quot;
}
">[ST17]</abbr>
* <abbr title="Ghosh - Distributed systems: an algorithmic approach (2006)" data-bibtex="@book{ghosh2006distributed,
    author = &quot;Ghosh, Sukumar&quot;,
    title = &quot;Distributed systems: an algorithmic approach&quot;,
    year = &quot;2006&quot;,
    publisher = &quot;Chapman and Hall/CRC&quot;
}
">[G06, Section IV]</abbr> "Faults and Fault-Tolerant Systems"
* <abbr title="Kshemkalyani and Singhal - Distributed computing: principles, algorithms, and systems (2011)" data-bibtex="@book{kshemkalyani2011distributed,
    author = &quot;Kshemkalyani, Ajay D and Singhal, Mukesh&quot;,
    title = &quot;Distributed computing: principles, algorithms, and systems&quot;,
    year = &quot;2011&quot;,
    publisher = &quot;Cambridge University Press&quot;
}
">[KS11]</abbr>
  * §13 Checkpointing and rollback recovery
* <abbr title="Lynch - Distributed algorithms (1996)" data-bibtex="@book{lynch1996distributed,
    author = &quot;Lynch, Nancy A&quot;,
    title = &quot;Distributed algorithms&quot;,
    year = &quot;1996&quot;,
    publisher = &quot;Elsevier&quot;
}
">[L96]</abbr>
  * § 21.3 "A Randomized Algorithm"
  * § 21.6 "Approximate Agreement"

mutual exclusion, i.e. distributed locks: according to
<abbr title="Steen and Tanenbaum - Distributed systems (2017)" data-bibtex="@book{van2017distributed,
    author = &quot;Van Steen, Maarten and Tanenbaum, Andrew S&quot;,
    title = &quot;Distributed systems&quot;,
    year = &quot;2017&quot;,
    publisher = &quot;Maarten van Steen Leiden, The Netherlands&quot;
}
">[ST17, §5.3.6]</abbr>, in practice this is done via a centralized server which is equivalent
to postgres locking w/ `idle_in_transaction_session_timeout`.
However, to avoid holding a database connection, specialized services like [etcd](https://etcd.io/docs/v3.6/dev-guide/api_concurrency_reference_v3/) can be used.
Alternatively, use a database table with a unique `lock_name` column. Acquire by repeatedly connecting and attempting to insert, release by deleting.

according to <abbr title="Kleppmann - Designing data-intensive applications: The big ideas behind reliable, scalable, and maintainable systems (2017)" data-bibtex="@book{kleppmann2017designing,
    author = &quot;Kleppmann, Martin&quot;,
    title = &quot;Designing data-intensive applications: The big ideas behind reliable, scalable, and maintainable systems&quot;,
    year = &quot;2017&quot;,
    publisher = {&quot; O&#x27;Reilly Media, Inc.&quot;}
}
">[K17, p336-...]</abbr>,
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
* <abbr title="Chandra and Hadzilacos and Toueg - The weakest failure detector for solving consensus (1996)" data-bibtex="@article{cht,
    author = &quot;Chandra, Tushar Deepak and Hadzilacos, Vassos and Toueg, Sam&quot;,
    title = &quot;The weakest failure detector for solving consensus&quot;,
    year = &quot;1996&quot;,
    issue_date = &quot;July 1996&quot;,
    publisher = &quot;Association for Computing Machinery&quot;,
    address = &quot;New York, NY, USA&quot;,
    volume = &quot;43&quot;,
    number = &quot;4&quot;,
    issn = &quot;0004-5411&quot;,
    url = &quot;https://doi.org/10.1145/234533.234549&quot;,
    doi = &quot;10.1145/234533.234549&quot;,
    journal = &quot;J. ACM&quot;,
    month = &quot;jul&quot;,
    pages = &quot;685–722&quot;,
    numpages = &quot;38&quot;,
    keywords = &quot;processor failures, partial synchrony, message passing, fault-tolerance, failure detection, crash failures, consensus problem, commit problem, atomic broadcast, asynchronous systems, agreement problem, Byzantine Generals&#x27; problem&quot;
}
">[CHT96]</abbr>

a different failure detector -> consensus
* in any environment
* <abbr title="Delporte-Gallet and Fauconnier and Guerraoui et al - The weakest failure detectors to solve certain fundamental problems in distributed computing (2004)" data-bibtex="@inproceedings{weakest,
    author = &quot;Delporte-Gallet, Carole and Fauconnier, Hugues and Guerraoui, Rachid and Hadzilacos, Vassos and Kouznetsov, Petr and Toueg, Sam&quot;,
    title = &quot;The weakest failure detectors to solve certain fundamental problems in distributed computing&quot;,
    year = &quot;2004&quot;,
    isbn = &quot;1581138024&quot;,
    publisher = &quot;Association for Computing Machinery&quot;,
    address = &quot;New York, NY, USA&quot;,
    url = &quot;https://doi.org/10.1145/1011767.1011818&quot;,
    doi = &quot;10.1145/1011767.1011818&quot;,
    booktitle = &quot;Proceedings of the Twenty-Third Annual ACM Symposium on Principles of Distributed Computing&quot;,
    pages = &quot;338–346&quot;,
    numpages = &quot;9&quot;,
    keywords = &quot;consensus, non-blocking atomic commit, quittable consensus, register, weakest failure detector&quot;,
    location = &quot;St. John&#x27;s, Newfoundland, Canada&quot;,
    series = &quot;PODC &#x27;04&quot;
}
">[DFG+04]</abbr>

a complicated failure detector -> distributed transaction
* <abbr title="Delporte-Gallet and Fauconnier and Guerraoui et al - The weakest failure detectors to solve certain fundamental problems in distributed computing (2004)" data-bibtex="@inproceedings{weakest,
    author = &quot;Delporte-Gallet, Carole and Fauconnier, Hugues and Guerraoui, Rachid and Hadzilacos, Vassos and Kouznetsov, Petr and Toueg, Sam&quot;,
    title = &quot;The weakest failure detectors to solve certain fundamental problems in distributed computing&quot;,
    year = &quot;2004&quot;,
    isbn = &quot;1581138024&quot;,
    publisher = &quot;Association for Computing Machinery&quot;,
    address = &quot;New York, NY, USA&quot;,
    url = &quot;https://doi.org/10.1145/1011767.1011818&quot;,
    doi = &quot;10.1145/1011767.1011818&quot;,
    booktitle = &quot;Proceedings of the Twenty-Third Annual ACM Symposium on Principles of Distributed Computing&quot;,
    pages = &quot;338–346&quot;,
    numpages = &quot;9&quot;,
    keywords = &quot;consensus, non-blocking atomic commit, quittable consensus, register, weakest failure detector&quot;,
    location = &quot;St. John&#x27;s, Newfoundland, Canada&quot;,
    series = &quot;PODC &#x27;04&quot;
}
">[DFG+04]</abbr>

eventually perfect failure detector =
* Every failed node is eventually detected as failed
* Eventually, no correct node is detected as failed

eventually perfect failure detector -> leader election
* Pick highest ID correct node as leader
* https://ucbrise.github.io/cs262a-spring2018/notes/09-ds-overview-consensus.pdf

total order broadcast <-> consensus
* <abbr title="Défago and Schiper and Urbán - Total order broadcast and multicast algorithms: Taxonomy and survey (2004)" data-bibtex="@article{defago,
    author = &quot;D\&#x27;{e}fago, Xavier and Schiper, Andr\&#x27;{e} and Urb\&#x27;{a}n, P\&#x27;{e}ter&quot;,
    title = &quot;Total order broadcast and multicast algorithms: Taxonomy and survey&quot;,
    year = &quot;2004&quot;,
    issue_date = &quot;December 2004&quot;,
    publisher = &quot;Association for Computing Machinery&quot;,
    address = &quot;New York, NY, USA&quot;,
    volume = &quot;36&quot;,
    number = &quot;4&quot;,
    issn = &quot;0360-0300&quot;,
    url = &quot;https://doi.org/10.1145/1041680.1041682&quot;,
    doi = &quot;10.1145/1041680.1041682&quot;,
    journal = &quot;ACM Comput. Surv.&quot;,
    month = &quot;dec&quot;,
    pages = &quot;372–421&quot;,
    numpages = &quot;50&quot;,
    keywords = &quot;Distributed systems, agreement problems, atomic broadcast, atomic multicast, classification, distributed algorithms, fault-tolerance, global ordering, group communication, message passing, survey, taxonomy, total ordering&quot;
}
">[DSU04, Sec. 2.4.5]</abbr>

total order broadcast <-> replicated state machine

consensus !-> distributed transaction
* <abbr title="Guerraoui - Revisiting the relationship between non-blocking atomic commitment and consensus (1995)" data-bibtex="@InProceedings{guerraoui,
    author = &quot;Guerraoui, Rachid&quot;,
    editor = &quot;H{\&#x27;e}lary, Jean-Michel and Raynal, Michel&quot;,
    title = &quot;Revisiting the relationship between non-blocking atomic commitment and consensus&quot;,
    booktitle = &quot;Distributed Algorithms&quot;,
    year = &quot;1995&quot;,
    publisher = &quot;Springer Berlin Heidelberg&quot;,
    address = &quot;Berlin, Heidelberg&quot;,
    pages = &quot;87--100&quot;,
    isbn = &quot;978-3-540-44783-2&quot;
}
">[G95]</abbr>
