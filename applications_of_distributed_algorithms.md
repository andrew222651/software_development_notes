# Applications of Distributed Algorithms

Two major application areas of distributed algorithms are distributed databases and fault tolerance in server-side applications.

## Fault tolerance in server-side applications

If you have to replicate data across a local and a remote database, it can be
helpful to store the local primary key in metadata associated with the remote data record.
This way, e.g. if rows can be deleted, we know if the remote data is associated with the current local instance of the data.

To avoid race conditions while working with distributed state, use a distributed lock, or concurrency control at the [task queue level](https://procrastinate.readthedocs.io/en/stable/howto/advanced/locks.html)

The order of operations can matter, e.g. suppose you have a row replicated across a
local and remote database and you want to delete both. If you can handle only the local row
existing but not only the remote row existing, delete the remote row first. 
If the process crashes after that, the system will not be in a broken state.

Suppose you use the stripe API to send a payment and you want it to only
happen once. In practice, keep sending it along with an idempotency key until
confirmation is received

When saving data from a remote service, it's possible that the data
has become stale between fetching it and saving it. When saving,
check to make sure that a newer version hasn't come in.

Suppose you have a task queue and you want tasks to run exactly once.
so deciding to commit is not the issue, it's figuring out what to do if the
committer crashes before reporting that the work is done. in
<abbr title="Ghosh - Distributed systems: an algorithmic approach (2006)" data-bibtex="@book{ghosh2006distributed,
    author = &quot;Ghosh, Sukumar&quot;,
    title = &quot;Distributed systems: an algorithmic approach&quot;,
    year = &quot;2006&quot;,
    publisher = &quot;Chapman and Hall/CRC&quot;
}
">[G06, ch14.6]</abbr> the idea is that if the work is a transaction,
if failure is detected, the tx can be rolled back to the latest saved checkpoint
and resumed. so maybe the worker starts a tx, stores the tx number (assumed
durable), then carries on. if it crashes we can look at the tx and see whether
it committed (if there isn't one saved the tx didn't get going).
<abbr title="Steen and Tanenbaum - Distributed systems (2017)" data-bibtex="@book{van2017distributed,
    author = &quot;Van Steen, Maarten and Tanenbaum, Andrew S&quot;,
    title = &quot;Distributed systems&quot;,
    year = &quot;2017&quot;,
    publisher = &quot;Maarten van Steen Leiden, The Netherlands&quot;
}
">[ST17, §8.3.2]</abbr> gives some terminology.
See also <https://docs.dbos.dev/python/tutorials/step-tutorial#transactions>.
the algorithm in <abbr title="Lee and Park and Kejriwal et al - Implementing linearizability at large scale and low latency (2015)" data-bibtex="@inproceedings{lee2015implementing,
    author = &quot;Lee, Collin and Park, Seo Jin and Kejriwal, Ankita and Matsushita, Satoshi and Ousterhout, John&quot;,
    title = &quot;Implementing linearizability at large scale and low latency&quot;,
    booktitle = &quot;Proceedings of the 25th Symposium on Operating Systems Principles&quot;,
    pages = &quot;71--86&quot;,
    year = &quot;2015&quot;
}
">[LPK+15]</abbr> also uses durable storage

garbage collection -- deleting data once all remote references are gone. "Broadly speaking, techniques for distributed GC fall under the same two paradigms as single-site
techniques: tracing, also known as marking, and reference tracking, a generalized form of reference counting." See [A survey of distributed garbage collection techniques](https://inria.hal.science/hal-01248224/file/SDGC_iwmm95.pdf)
