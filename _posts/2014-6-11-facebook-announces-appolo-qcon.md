---
layout: post
title: "Facebook announces Apollo at QCon NY 2014" 
category : conferences   
tags : [data, conferences]
excerpt: "Today I attended a talk ['How Facebook Scales Big Data Systems'](https://qconnewyork.com/presentation/how-facebook-scales-big-data-systems) at QCon NY 2014. It turns out that Jeff Johnson took this opportunity to announce a new project by Facebook: Apollo. The following is a rough transcript of the talk."

---

Apollo is a system Facebook has been working on for about a year in their New York office. The tagline of Apollo is 'consistency at scale'. Its design is heavily influenced by their experience with HBase.

![Jeff Johnson](/pics/qcon_apollo1.jpg)

### Why?

Facebook generally favors CP systems. They have four datacenters using master/slave style replication. Missing the 'A'vailability usually isn't a big deal within a datacenter, but across DCs it becomes troublesome.

The wishlist that lead to Apollo is roughly as follows:

* we need some kind of transactions
* acked writes should be eventually visible and not lost
* availability 

So the question really boils down to: can we layer AP on top of CP?

### Apollo
Apollo itself is written in 100% C++11, also using thrift2. The design is based on having a hierarchy of shards. These form the basic building block of Apollo. Essentially like HDFS (regions) are the building block for HBase. It supports thousands of shards, scaling from 3 servers to ~10k servers.

The shards use Paxos style quorum protocols (CP). Raft is used for consensus. Fun sidenote: this turned out to be not much simpler than multi-paxos, even though that was the expectation.

RocksDB (a key-val store, log-structured storage) or MySQL can be used as underlying storage. The storage primitives offered by Apollo are:

* binary value
* map
* pqueue
* CRDTs


### Use cases

The apparent sweetspot for Apollo is online, low-latency storage of aforementioned data structures. Especially where atomic transformations of these data structures are required. The size of 'records' should range from a byte to about a megabyte.

Every operation on the Apollo api is atomic. For exampe, you can pass in conditions into a read/write call. As an example, a condition can assert that a map contains a certain key. The operation will only go through if the condition (atomically) holds.

![Jeff Johnson](/pics/qcon_apollo2.jpg)
_First write call mimics a traditional last-write wins KV store. Second one does optimistic concurrency, last one allows for complex transactional behavior. Note that a write can also return (consistent) reads!_

Atomicity works across shards, at varying levels:

![Jeff Johnson](/pics/qcon_apollo3.jpg)

The idea is that Apollo provides CP locally and AP between remote sites.

### Fault tolerant state machines
In addition to just storing and querying data, Apollo has another unique feature: it allows execution of user submitted code in the form of state machines. These fault tolerant state machines are primarily used for Apollo's internal implementation. E.g. to do shard creation/destruction, load balancing, data migration, coordinating cross-shard transactions.

But users can submit FTSMs too. These are persistently stored, so it tolerates node failures. A shard owns a FTSM. A state machine may have side effects (e.g. call an external API), but all state changes are submitted through the shard replication mechanism. The exact way to create these FTSMs were not discussed.

### Applications of Apollo at Facebook
One of the current applications of Apollo at Facebook is as a reliable in-memory database. For this use case it is setup with a RAFT write-ahead-log on a Linux tmpfs. They replace memcache(d) with this setup, gaining transactional support in the caching layer.

Furthermore it powers TACO, an in-memory version of their TAO graph. It supports billions of reads per second and millions of writes this way.

A more persistent application of Apollo is to provide reliable queues, for example to manage the push notifications.

## Open source?
Currently, Apollo is developed internally at Facebook. No firm claims were made during the talk that it will be opensourced. It was mentioned as a possibility after internal development settles down.

This was a quick write-up, twenty minutes after the talk ended. Any factual errors are solely caused by me misinterpreting or mishearing stuff. I expect more information to trickle out of Facebook engineering soon. That said, I hope this post paints an adequate initial picture of Facebook's Apollo!

