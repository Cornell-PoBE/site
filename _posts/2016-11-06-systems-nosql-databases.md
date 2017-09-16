---
layout: post
title: "Systems and NoSQL Databases"
---

# (1 Week)

# Introduction
In this section of the course we will be doing a survey into data storage beyond the relational model and RDBMS.

* "NoSQL" systems (DynamoDB, Cassandra, HBase, MongoDB, Couchbase, ...)
* New processing models (MapReduce, Spark...)

Starting in mid-2000s, custom data storage solutions developed in industry that include and are not limited to: Amazon DynamoDB, Google Bigtable, Facebook Cassandra, etc. Now these solutions are available for general use in various ports/variants.

Sometimes an RDBMS is more than we need as we do not really care so much about expressive queries or transactional guarantees. An example application would include: processing internet logs where append/retrieve operations are essential but complex queries, transactions, etc are less so. Other reasons to move to noSQL are for horizontal scalability. As cheap commodity machines are becoming more and more common, the need to scale horizontally instead of vertically (where one needs to buy bigger, and fancier servers) is seemingly the most economical.

Other problems that you are likely to encounter is the issue for when data has lots of NULL values. Data may not be as complete as the relational model that is forced to fit under. As such there could be a case where there are many "missing" (optional) attributes. This would mean that there would be NULLs all over the place which would be quite painstaking considering the complex ISA hierarchy that you established when building out the relational model. Here is an example schema:

|          Wine         	|       Region       	| Parker Score 	|
|:---------------------:	|:------------------:	|--------------	|
|       Pinot Noir      	| Williamette Valley 	|      90       |
|         Merlot        	|   Saint-Emillion   	|      95       |
| "Ilans Tutti Fruitti" 	|   Ilan's Backyard  	|     NULL      |

In such a system if Ilan's Backyard were to make a bunch of wines, the chances of Richard Parker immediately giving a score is slim. As such, there would be MANY missing values in this type of relational model.


The other issue is that relational schemas are hard to change, especially if you frequently add new fields to your data. Let's say Ilan's Backyard would like to add "style of music" to express the optimal music to play when drinking a particular wine. The addition of this new field would be quite expensive to add to the entire relational model, despite my eagerness to add more information towards describing my wine:

|          Wine         	|       Region       	| Parker Score 	| Music Pairing 	|
|:---------------------:	|:------------------:	|--------------	|---------------	|
|       Pinot Noir      	| Williamette Valley 	| 90           	| NULL          	|
|         Merlot        	|   Saint-Emillion   	| 95           	| NULL          	|
| "Ilans Tutti Fruitti" 	|   Ilan's Backyard  	| NULL         	| Swing         	|
| "Ilans Deep Red"      	| Ilan's Backyard    	| NULL         	| Bachata       	|

This is where our exploration begins. What does data look like if it is not relational?
* More complex and/or flexible than relations (Trees / Nested Document Structure)
* Simpler than relations (Key-Value pair)
* LOTS (you are encouraged to explore!)

The field of NoSQL covers all of the above.

We will now take a deep dive into the topics below:

## Data Storage Solutions
### Key Value Stores
Sometimes our data doesn't have a complex structure and a very simple data model will do just fine (i.e.) pure key-value model.
Data in such stores consist of <key,value> pairs where the key is some identifier, value can be anything, and the data model itself is completely schema-less. If we look above, this addresses the issues of not worrying about an over-abundance of NULLs and adding fields "willy nilly." The values themselves can have some form of internal structure, but datastore doesn't really "know about it" and you can't query on it.
#### Project Voldemort
This database was originally developed at LinkedIn and is now open source.
The API is simply the following:
```python
value = store.get(key)
store.put(key,value)
store.delete(key)
```
To put this simply, it is basically just a "big, distributed, persistent, fault-tolerant hash table". This database is heavily optimized for performance on those above, basic operations. As such, a pure, and quite simple, key-value model can be all you need, but that API can be quite constraining in some use cases (i.e. what about range queries??) Let's see other system's build upon this:
#### Redis
This key-value store allows you to have values of various types: strings, lists, hashmaps, etc. Redis, in comparison to Voldemort, has a richer API than get/put. For example:
```bash
redis> EXISTS mykey
(integer) 0
redis> APPEND mykey "Hello"
(integer) 5
redis> APPEND mykey "World"
(integer) 11
redis> GET mykey
"Hello World"
redis> SET mykey "This is a string"
OK
redis> GETRANGE mykey 0 3
"THIS"
redis> LPUSH mylist "world"
(integer) 1
redis> LPUSH mylist "hello"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
```
Redis is an open source, in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperlog and geospatial indexes with radius queries. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.
#### DynamoDB
**Introduction**

Amazon runs a world-wide e-commerce platform that serves tens
of millions customers at peak times using tens of thousands of
servers located in many data centers around the world. There are
strict operational requirements on Amazon’s platform in terms of
performance, reliability and efficiency, and to support continuous
growth the platform needs to be highly scalable. Reliability is one
of the most important requirements because even the slightest
outage has significant financial consequences and impacts
customer trust. In addition, to support continuous growth, the
platform needs to be highly scalable.  

DynamoDB uses a synthesis of well known techniques to achieve
scalability and availability: Data is partitioned and replicated
using consistent hashing which will be elaborated upon below, and consistency is facilitated by object versioning. The consistency among replicas during updates is maintained by a quorum-like technique and a
decentralized replica synchronization protocol. DynamoDB employs
a gossip-based distributed failure detection and membership
protocol. DynamoDB is a completely decentralized system with
minimal need for manual administration. Storage nodes can be
added and removed from DynamoDB without requiring any manual
partitioning or redistribution.

In comparison to Voldemort and Redis, DynamoDB Looks a bit more like a relational DB as the model is based on `Tables.`
These tables contain items ("tuples") that each have an attribute that includes a primary key and attributes that have values.
A single attribute may have more than one value. As such, this database supports both document and key-value store models.

For example:

A ProductCatalog example:
```bash
{
   ID = 101
   ProductName = "Book 101 Title"
   ISBN = "111-1111111111"
   Authors = [ "Author 1", "Author 2" ]
   Price = -2
   Dimensions = "8.5 x 11.0 x 0.5"
   PageCount = 500
   InPublication = 1
   ProductCategory = "Book"
}
```

As such, even though these tables have attributes, they are not really designed to support arbitrary selection queries, but they do give you a lot of power with the keys your items can have. The keys that we have mostly investigated in RDBMS world were primary keys.

In DynamoDB primary keys can be simple or composite
* Simple (just one attribute): DynamoDB builds a hash index so that you may do equality selections fast
* Composite key has two parts – partition key and sort key. Where you have a hash index on first, range index on second. These range indexes can be used for efficient queries but on this attribute only

Composite Key example:

|          Table Name         	        |       Partition Key     | Sort Key  	  |
|:-------------------------------------:|:-----------------------:|:-------------:|
| Forum (Name, ...)                     | Name                    |      -        |
| Thread (ForumName, Subject, ...)      | ForumName               | Subject       |
| Reply (Id, ReplyDateTime, ...)        | Id                      | ReplyDateTime |

For the given example it is therefore optimized for queries that find all replies for a given forum/ thread. As such you would choose to encode forum and thread in Id i.e. Reply Id has the format `"Forum1#Thread1"`

The DynamoDB API clearly shows the advancement from Voldemort and Redis:

* Create/Update/Delete/Describe Table
* GetItem (specify primary key)
* PutItem (creating new item)
* Option to create only if doesn't already exist (overwritten by default)
* Update/Delete Item (specify by primary key)
* Query (retrieve everything that matches a given value for the partition key)
* Filter Query (based on a range on the sort key [must specify value of partition key first])
* Table Scans

**Secondary Indexes**

A local secondary index allows you to specify an alternate sort key where the partition key stays the same.
A global secondary indexes allow different partition key too. However, you are allowed only up to 5 indexes for each kind of table. All indexes must be created up front at table creation time. As such, you need to think about your queries up front at table creation, not as a final tuning step. As such, it is a common feature in NoSQL design to optimize your table design based on your queries. Your queries might even expect you to fully de-normalize your data (pre-join).

#### Extended analysis of DynamoDB and common distributed principles
The problems that DynamoDB faces and its respective solutions are represented in the table below. We encourage you to spend your time reading the following [paper](http://www.cs.utexas.edu/users/lorenzo/corsi/cs380d/papers/dynamo.pdf) to investigate these solutions on your own time. Classes like CS 4320 and [CS 5414](http://www.cs.cornell.edu/courses/cs5414/2016fa/) investigate such topics and we highly encourage you to take these classes to further immerse yourself in the world of Distributed Systems.

|          Problem         	        |       Technique     | Advantage 	|
|:---------------------------------:|:------------------:	|--------------	|
|       Partitioning      	        | Consistent Hashing  | Incremental Scalability  |
| High Availability for writes      |   Vector clocks with reconciliation during reads | Version size is decoupled from update rates.|
| Handling temporary failures  	    | Sloppy Quorum and hinted handoff   	| Provides high availability and durability guarantee when some of the replicas are not available.         	|
| Recovering from permanent failures| Anti-entropy using Merkle trees     	| Synchronizes divergent replicas in the background.          	|
| Membership and failure detection  | Gossip-based membership protocol and failure detection | Preserves symmetry and avoids having a centralized registry for storing membership and node liveness information |


The following topics utilize different principles in Distributed Systems and are leveraged in various NoSQL databases.

**Consistent Hashing**

Dynamo’s partitioning scheme relies on consistent hashing to distribute the load across multiple storage hosts. In consistent hashing, the output range of a hash function is treated as a fixed circular space or “ring” (i.e. the largest hash value wraps around to the smallest hash value). Each node in the system is assigned a random value within this space which represents its “position” on the ring. Each data item identified by a key is assigned to a node by hashing the data item’s key to yield its position on the ring, and then walking the ring clockwise to find the first node with a position larger than the item’s position.

Thus, each node becomes responsible for the region in the ring between it and its predecessor node on the ring. The principle advantage of consistent hashing is that departure or arrival of a node only affects its immediate neighbors
![Consistent-Hashing](https://i.gyazo.com/62803f773387b4c6e425e099bd845f82.png)

**Versioning**

Mimicking the structure of how you would expect Git to handle merge conflicts. DynamoDB uses something called vector clocks in order to capture causality between different versions of the same object. A vector clock is effectively a list of (node, counter) pairs. One vector clock is associated with every version of every object. One can determine whether two versions of an object are on parallel branches or have a causal ordering, by examine their vector clocks. If the counters on the first object’s clock are less-than-or-equal to all of the nodes in the second clock, then the first is an ancestor of the second and can be forgotten. Otherwise, the two changes are considered to be in conflict and require reconciliation. An example is shown below where we have three different processes: `Sx`, `Sy`, and `Sz`, each writing values which need to be reconciled.

![Vector-Clocks](https://i.gyazo.com/c0832308af8a2a3167d6b6ccf705c59d.png)

**Quorums**

A quorum is the minimum number of votes that a distributed transaction has to obtain in order to be allowed to perform an operation in a distributed system i.e. a set of 6 machines need at least 3 machines to confirm that they are writing before the write is approved and executed by the system. A quorum-based technique is implemented to enforce consistent operation in a distributed system. However, in the case of DynamoDB, if it uses a traditional quorum approach it would be unavailable during server failures and network partitions, and
would have reduced durability even under the simplest of failure conditions. To remedy this, as DynamoDB prefers durability, it does not enforce strict quorum membership and instead uses a “sloppy quorum”; all read and write operations are performed on the first N healthy nodes from the preference list, which may not always be the first N nodes encountered while walking the consistent hashing ring. For example, let us look at the consistent-hashing ring above and assume a "sloppy quorum" of size N=3. In this example, if node A is temporarily down or unreachable during a write operation then a replica that would normally have lived on A will now be sent to node D. This is done to maintain the desired availability and durability guarantees. The replica sent to D will have a hint in its metadata that suggests which node was the intended recipient of the replica (in this case A). Nodes that receive hinted replicas will keep them in a
separate local database that is scanned periodically. Upon detecting that A has recovered, D will attempt to deliver the replica to A. Once the transfer succeeds, D may delete the object from its local store without decreasing the total number of replicas in the system.
This form of "hinted handoff" allows DynamoDB to ensure that the read and write operations are not failed due to temporary node or network
failures.

**Replication**

To achieve high availability and durability, DynamoDB replicates its data on multiple hosts. Each data item is replicated on N hosts, where N represents a parameter configured “per-instance”. Each key, k, is assigned to a coordinator node.  The coordinator is in charge of the replication of the data items that fall within its range. In addition to locally storing each key within its range, the coordinator replicates these keys at the N-1 clockwise successor nodes in the ring. This results in a system where each node is responsible for the region of the ring between it and its Nth predecessor. In the above ring figure, node B replicates the key k at nodes C and D in addition to storing it locally. Node D will store the keys that fall in the ranges `(A, B]`, `(B, C]`, and `(C, D]`.

**Gossip**

A gossip protocol is a style of computer-to-computer communication protocol inspired by the form of gossip seen in social networks. Modern distributed systems often use gossip protocols to solve problems that might be difficult to solve in other ways, either because the underlying network has an inconvenient structure, is extremely large, or because gossip solutions are the most efficient ones available. The term epidemic protocol is sometimes used as a synonym for a gossip protocol, because gossip spreads information in a manner similar to the spread of a virus in a biological community. In Amazon’s environment node outages (due to failures and maintenance tasks) are often transient but may last for extended intervals. A node outage rarely signifies a permanent departure and therefore should not result in rebalancing of the partition assignment or repair of the unreachable replicas. Similarly, manual error could result in the unintentional startup of new nodes. For these reasons, it was deemed appropriate to use an explicit mechanism to initiate the addition and removal of nodes from the consistent hashing ring. An administrator uses a command line tool or a browser to connect to a node and issue a membership change to join a node to a ring or remove a node from a ring. The node that serves the request writes the membership change and its time of issue to persistent store. The membership changes form a history because nodes can be removed and added back multiple times. A gossip-based protocol propagates membership changes and maintains an eventually consistent view of membership. Each node contacts a peer chosen at random every second and the two nodes efficiently reconcile their persisted membership change histories.

#### Google App Engine Datastore
Google App Engine Datastore is a NoSQL document database built for automatic scaling, high performance, and ease of application development that also extends the key-value store. GAE Datastore contains entities ("tuples"), which have some properties ("attributes") and belong to kinds ("tables") where you can perform programmatic queries (basically select-project) but have to create indexes on appropriate properties to perform queries on non-key values.

* GAE Datastore is designed to automatically scale to very large data sets, allowing applications to maintain high performance as they receive more traffic

* GAE Datastore writes scale by automatically distributing data as necessary.

* Cloud Datastore reads scale because the only queries supported are those whose performance scales with the size of the result set (as opposed to the data set). This means that a query whose result set contains 100 entities performs the same whether it searches over a hundred entities or a million. This property is the key reason some types of queries are not supported.

### Column Family data model

CAP Theorem

ACID

#### Apache Cassandra
#### Apache HBase

## Document-oriented DB
### MongoDB

Data model

Serialization (Avro / Parquet / Protobuf)

Querying

## Map Reduce
## Distributed Systems
Distributed computing is a field of computer science that studies distributed systems. A distributed system is a model in which components located on networked computers communicate and coordinate their actions by passing messages. The components interact with each other in order to achieve a common goal. Three significant characteristics of distributed systems are: concurrency of components, lack of a global clock, and independent failure of components. There are many alternatives for the message passing mechanism, including pure HTTP like we have seen, RPC-like connectors, and message queues which could be in the form of a Pub-Sub system that we will elaborate on below.
### Message Passing
Message-oriented middleware (MOM) is software or hardware infrastructure supporting sending and receiving messages between distributed systems. MOM allows application modules to be distributed over heterogeneous platforms and reduces the complexity of developing applications that span multiple operating systems and network protocols. The middleware creates a distributed communications layer that insulates the application developer from the details of the various operating systems and network interfaces. APIs that extend across diverse platforms and networks are typically provided by a MOM. MOM provides software elements that reside in all communicating components of a client/server architecture and typically support asynchronous calls between the client and server applications. MOM reduces the involvement of application developers with the complexity of the master-slave nature of the client/server mechanism.
#### Types of middleware
* Remote Procedure Call or RPC-based middleware
* Object Request Broker or ORB-based middleware
* Message Oriented Middleware or MOM-based middleware
All these models make it possible for one software component to affect the behavior of another component over a network. They are different in that RPC and ORB-based middleware create systems of tightly coupled components, whereas MOM-based systems allow for a looser coupling of components. In essence, RPC and ORB-based systems are synchronous, where when one procedure calls another, it must wait for the called procedure to return before it can do anything else.

Central reasons for using a message-based communications protocol include its ability to store/buffer, route, or transform messages while conveying them from senders to receivers. These message passing systems engage in certain conceptual models that deal with concurrent computation. An example of such a model is the Actor Model.

#### Actor Pattern
The actor model is a conceptual model to deal with concurrent computation. It defines some general rules for how the system’s components should behave and interact with each other. The most famous language that uses this model is probably `Erlang`, but it is also leveraged by `Scala` in the async library called `Akka`.

An actor is the primitive unit of computation. It’s the thing that receives a message and does some kind of computation based on it.

The idea is very similar to what we have in object-oriented languages: An object receives a message (a method call) and does something depending on which message it receives (which method we are calling).
The main difference is that actors are completely isolated from each other and they will never share memory. It’s also worth noting that an actor can maintain a private state that can never be changed directly by another actor. In the actor model everything is an actor and they need to have addresses so one actor can send a message to another. Although multiple actors can run at the same time, an actor will process a given message sequentially. This means that if you send 3 messages to the same actor, it will just execute one at a time. To have these 3 messages being executed concurrently, you need to create 3 actors and send one message to each.

Messages are sent asynchronously to an actor, that needs to store them somewhere while it’s processing another message. The mailbox is the place where these messages are stored.
![Actor-Model](http://www.brianstorti.com/assets/images/actors.png)

The two main advantages of such a model are for fault tolerance and distribution.

**Fault Tolerance**
In such a model the actors are processes are completely isolated, meaning its state is not going to influence any other process. As such, faults can be managed by having a supervisor, that is basically another process, that will be notified when the supervised process crashes and then can do something about it.

This makes it possible to create systems that “self heal”, meaning that if an actor gets to an exceptional state and crashes, by whatever reason, a supervisor can do something about it to try to put it in a consistent state again (and there are multiple strategies to do that, the most common being just to restart the actor with its initial state).

**Distribution**
Naturally these types of models lend themselves towards distribution where it doesn’t matter if the actor that I’m sending a message to is running locally or in another node. For example if an actor is just a process with a mailbox and an internal state, and it just respond to messages, who cares in which machine it’s actually running? As long as we can make the message get there we are fine. This allows us to create systems that leverage multiple computers and helps us to recover if one of them fail.

#### Back to Middleware
Another advantage of messaging provider mediated messaging between clients is that by adding an administrative interface, you can monitor and tune performance. Client applications are thus effectively relieved of every problem except that of sending, receiving, and processing messages. It is up to the code that implements the MOM system and up to the administrator to resolve issues like interoperability, reliability, security, scalability, and performance.
I will now go into some advantages and disadvantages in using a MOM rather than the standard HTTP or RPC systems we commonly use.
#### Advantages for MOM
* Asynchronicity: a client makes an API call to send a message to a destination managed by the provider. The call invokes provider services to route and deliver the message. Once it has sent the message, the client can continue to do other work, confident that the provider retains the message until a receiving client retrieves it. The message-based model, coupled with the mediation of the provider, makes it possible to create a system of loosely coupled components.
* Routing: many message-oriented middleware implementations depend on a message queue system. Some implementations permit routing logic to be provided by the messaging layer itself, while others depend on client applications to provide routing information or allow for a mix of both paradigms. Some implementations make use of broadcast or multicast distribution paradigm
* Transformation: in a message-based middleware system, the message received at the destination need not be identical to the message originally sent. A MOM system with built-in intelligence can transform messages en route to match the requirements of the sender or of the recipient
#### Disadvantages for MOM
The primary disadvantage of many message-oriented middleware systems is that they require an extra component in the architecture, the message transfer agent (message broker). As with any system, adding another component can lead to reductions in performance and reliability, and can also make the system as a whole more difficult and expensive to maintain.

**Message Brokers**
Some commonly used message brokers include: `Apache Kafka`,`Celery`, and `RabbitMQ`. These message brokers are usually based on a pub-sub (publish-subscribe) pattern. This messaging pattern, different from the client-server one we learned in Lecture 1, is where senders of messages, called publishers, do not program the messages to be sent directly to specific receivers, called subscribers, but instead categorize published messages into classes without knowledge of which subscribers, if any, there may be. Similarly, subscribers express interest in one or more classes and only receive messages that are of interest, without knowledge of which publishers, if any, there are.

In the publish–subscribe model, subscribers typically receive only a subset of the total messages published. The process of selecting messages for reception and processing is called filtering. There are two common forms of filtering: topic-based and content-based.

* topic-based system: messages are published to "topics" or named logical channels. Subscribers in a topic-based system will receive all messages published to the topics to which they subscribe, and all subscribers to a topic will receive the same messages. The publisher is responsible for defining the classes of messages to which subscribers can subscribe.

* content-based system: messages are only delivered to a subscriber if the attributes or content of those messages match constraints defined by the subscriber. The subscriber is responsible for classifying the messages.

Some systems support a hybrid of the two; publishers post messages to a topic while subscribers register content-based subscriptions to one or more topics.

These systems are commonly used for logging purposes but message passing in distributed systems are an example use case.

**Messaging Queues**
The message queue paradigm is a sibling of the pub-sub pattern. Message queues provide an asynchronous communications protocol, meaning that the sender and receiver of the message do not need to interact with the message queue at the same time. Messages placed onto the queue are stored until the recipient retrieves them. Message queues have implicit or explicit limits on the size of data that may be transmitted in a single message and the number of messages that may remain outstanding on the queue. `StormMQ`, and `Apache Quid` are both examples.

### Why use Distributed Systems?
Well, typically, data in a large database may be distributed across multiple machines/nodes or replicated on multiple machines/nodes.

The main reasons to distribute your data would usually be for scalability or availability reasons, where the conditions could be that there are few programs that need to access ALL of your data and there is a need for strong data locality i.e. the customers need to be satisfied with faster response when given their geographical area.

The main reasons to replicate your data would be again for scalability or availability reasons, where you may want multiple machines to help share the load on very high-demand data or for fault-tolerance reasons where if a machine goes down the system will still function.

### Sharding
### Transactions
### Bloom Filters
