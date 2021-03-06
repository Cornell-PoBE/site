---
layout: post
title: "Systems and NoSQL Databases"
---

# (2 Weeks)

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

**Gossip for Fault Detection**

A gossip protocol is a style of computer-to-computer communication protocol inspired by the form of gossip seen in social networks. Modern distributed systems often use gossip protocols to solve problems that might be difficult to solve in other ways, either because the underlying network has an inconvenient structure, is extremely large, or because gossip solutions are the most efficient ones available. The term epidemic protocol is sometimes used as a synonym for a gossip protocol, because gossip spreads information in a manner similar to the spread of a virus in a biological community. In Amazon’s environment node outages (due to failures and maintenance tasks) are often transient but may last for extended intervals. A node outage rarely signifies a permanent departure and therefore should not result in rebalancing of the partition assignment or repair of the unreachable replicas. Similarly, manual error could result in the unintentional startup of new nodes. For these reasons, it was deemed appropriate to use an explicit mechanism to initiate the addition and removal of nodes from the consistent hashing ring. An administrator uses a command line tool or a browser to connect to a node and issue a membership change to join a node to a ring or remove a node from a ring. The node that serves the request writes the membership change and its time of issue to persistent store. The membership changes form a history because nodes can be removed and added back multiple times. A gossip-based protocol propagates membership changes and maintains an eventually consistent view of membership. Each node contacts a peer chosen at random every second and the two nodes efficiently reconcile their persisted membership change histories.

**CAP Theorem**

It is known that data is replicated for performance and failover purposes i.e. you want multiple machines to help share the load on very high-demand data or for fault-tolerance reasons where if a machine goes down the system will still function.

Therefore, it would be ideal that you want access to a single object to be atomic / linearizable i.e. if several processes are using the same object, it should "look like" they were accessing it in sequence on a single-node system. Linearizablity requirements include:
* total order on operations across the whole system
* consistent with wall-clock ordering for non-overlapping regions
* if a read is ordered after a write, the read should see the value written by that write (or a later one)

Linearizablity is also known as consistency. The problem is that we want to enforce this without sacrificing availability i.e. users can perform reads/writes when they want to.

With all this talk of consistency and being highly available it is important to introduce the CAP theorem as this is something that is used throughout distributed computing and when analyzing most NoSQL systems.

The CAP theorem, also named Brewer's theorem after computer scientist Eric Brewer, states that it is impossible for a distributed data store to simultaneously provide more than two out of the following three guarantees:
* Consistency: Every read receives the most recent write or an error
* Availability: Every request receives a (non-error) response – without guarantee that it contains the most recent write
* Partition Tolerance: The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes

It is also worth noting, that when breaking down the CAP theorem, it is actually possible to have all 3, but you can't have C(Consistency) and A(Availability) during P(network-failure).

Furthermore, no distributed system is safe from network failures, thus network partitioning generally has to be tolerated. In the presence of a partition, one is then left with two options: consistency or availability. When choosing consistency over availability, the system will return an error or a time-out if particular information cannot be guaranteed to be up to date due to network partitioning. When choosing availability over consistency, the system will always process the query and try to return the most recent available version of the information, even if it cannot guarantee it is up to date due to network partitioning.

CAP is frequently misunderstood as if one had to choose to abandon one of the three guarantees at all times. In fact, the choice is really between consistency and availability only when a network partition or failure happens ; at all other times, no trade-off has to be made.

![CAP-Venn](https://lh5.googleusercontent.com/-2QOlwUJQC1o/UsWOOFUTRkI/AAAAAAAAAEA/Y-gRB58QixY/w547-h520/cap_venn.png)

As such example systems like: MongoDB, Apache HBase, and Google Bigtable provide guarantees of enforced or Strong Consistency while systems like: CouchDB, Riak, and Apache Cassandra give eventual consistency. We will be analyzing HBase and Cassandra in the next section.

When looking at DynamoDB, since that is our current example, we see that it sacrifices consistency (on certain failure instances) for availability. As such this is a AP system that is highly available.

#### Google App Engine Datastore
Google App Engine Datastore is a NoSQL document database built for automatic scaling, high performance, and ease of application development that also extends the key-value store. GAE Datastore contains entities ("tuples"), which have some properties ("attributes") and belong to kinds ("tables") where you can perform programmatic queries (basically select-project) but have to create indexes on appropriate properties to perform queries on non-key values.

* GAE Datastore is designed to automatically scale to very large data sets, allowing applications to maintain high performance as they receive more traffic

* GAE Datastore writes scale by automatically distributing data as necessary.

* Cloud Datastore reads scale because the only queries supported are those whose performance scales with the size of the result set (as opposed to the data set). This means that a query whose result set contains 100 entities performs the same whether it searches over a hundred entities or a million. This property is the key reason some types of queries are not supported.

### Column Family data model
A column family is a NoSQL object that contains columns of related data. It is a tuple (pair) that consists of a key-value pair, where the key is mapped to a value that is a set of columns. In analogy with relational databases, a column family is as a "table", each key-value pair being a "row". Each column is a tuple (triplet) consisting of a column name, a value, and a timestamp. In a relational database table, this data would be grouped together within a table with other non-related data. A keyspace in a NoSQL data store is an object that holds together all column families of a design. The keyspace has similar importance like a schema has in a database. However, in contrast, the contents of the keyspace can be column families, each having different number of columns, or even different columns. So, the column families that somehow relate to the row concept in relational databases do not stipulate any fixed structure. The only point that is the same with a schema is that it also contains a number of "objects", which are tables in RDBMS systems and here column families or super columns. So, in distributed data stores, the whole burden to handle rows that may even change from data-store update to update lies on the shoulders of the programmers.

In essence, the basic idea is that there is a map with keys being: `<rowid, columnid, timestamp` and the values being, whatever is specified by the keyspace. It is similar to a normal relational table but it can have several different versions of the same data item with different timestamps. The functions of these timestamps is to keep multiple versions of the same data and to bound the number of versions kept. Naturally, the default queries will return the newest version. The use of column families are to design for when most rows do not have values in all the possible columns or where the map is very sparse. Similar to what was going on in DynamoDB, but more extreme.

Additional abstractions are that all columns store related data are in a single column family which could be defined as a container for columns. As such, it functions as the unit of storage and of access. However, these column families need to be specified in advance, columns, not necessarily.

Here is an example of the column family:
![Column-Family](https://i.imgur.com/uhpXToH.png)
Where you have two column families that will respectively relate to certain queries of focus.

We now will be looking at the two largest NoSQL stores that leverage the Column Family data model.

#### Apache Cassandra

Apache Cassandra is a free and open-source distributed NoSQL database management system designed to handle large amounts of data across many commodity servers, providing high availability with no single point of failure. Cassandra offers robust support for clusters spanning multiple datacenters, with asynchronous masterless replication allowing low latency operations for all clients.

Cassandra is essentially a hybrid between a key-value and a column-oriented (or tabular) database management system. Its data model is a partitioned row store with tunable consistency. Rows are organized into tables; the first component of a table's primary key is the partition key; within a partition, rows are clustered by the remaining columns of the key. Other columns may be indexed separately from the primary key. Tables may be created, dropped, and altered at run-time without blocking updates and queries.Cassandra cannot do joins or subqueries. Rather, Cassandra emphasizes denormalization through features like collections.

The main features of Apache Cassandra are the following:
* Decentralized: Every node in the cluster has the same role. There is no single point of failure. Data is distributed across the cluster (so each node contains different data), but there is no master as every node can service any request
* Replication: Replication strategies are configurable. Key features of Cassandra’s distributed architecture are specifically tailored for multiple-data center deployment, for redundancy, for failover and disaster recovery
* Scalability: Designed to have read and write throughput both increase linearly as new machines are added, with the aim of no downtime or interruption to applications
* Fault-tolerant: Data is automatically replicated to multiple nodes for fault-tolerance
* Tunable consistency: Writes and reads offer a tunable level of consistency, all the way from "writes never fail" to "block for all replicas to be readable", with the quorum level in the middle
* MapReduce support: Cassandra has Hadoop integration, with MapReduce support. (This is elaborated upon below)
* Query language: Leverages Cassandra Query Language (CQL) which is a simple interface for accessing Cassandra

Something that is extremely unique, in my opinion, is the tunable consistency.

**Eventual Consistency**

System must always be able to take reads and write even when network becomes partitioned. This comes at a consequence as you can no longer guarantee all replicas keep in sync. Therefore, replicas do eventually get back into sync, but there is a definite window of inconsistency.

From a system perspective there are 3 main parameters to focus on:
* N: replication factor i.e. How many copies of the data are stored?
* R: How many replicas are checked during a read?
* W: How many replicas must acknowledge receipt of a write?

If `W + R > N`, you can guarantee strong consistency where the read and write set always overlaps, but at the cost of availability. Here are some example analysis of trade-offs:
* `W = N`, `R = 1` achieves fastest reads
* `W = 1`, `R = N` achieves fastest writes
* `W=Q`, `R=Q` where `Q = N /(2 + 1)` where Q is a quorum or a consensus that is elaborated upon below in the Distributed Systems section.

With such tunable consistency you can balance the trade-offs with a system like Cassandra to determine when you want certain levels of consistency vs. availability by setting the appropriate `ConsistencyLevel`.

#### Apache HBase

HBase is a NoSQL distributed database modeled after Google's Bigtable and is written in Java. It is developed as part of Apache Software Foundation's Apache Hadoop project and runs on top of HDFS (Hadoop Distributed File System), providing Bigtable-like capabilities for Hadoop. That is, it provides a fault-tolerant way of storing large quantities of sparse data i.e. small amounts of information caught within a large collection of empty or unimportant data, such as finding the 50 largest items in a group of 2 billion records, or finding the non-zero items representing less than 0.1% of a huge collection.

HBase features compression, in-memory operation, and Bloom filters on a per-column basis as outlined in the original Bigtable paper. Tables in HBase can serve as the input and output for MapReduce jobs run in Hadoop, and may be accessed through the Java API but also through REST, `Avro` or `Thrift` gateway APIs. HBase is a column-oriented key-value data store and has been idolized widely because of its lineage with Hadoop and `HDFS`. HBase runs on top of HDFS and is well-suited for faster read and write operations on large datasets with high throughput and low input/output latency. HBase is not a direct replacement for a classic SQL database, however the `Apache Phoenix` project provides an SQL layer for HBase as well as JDBC driver that can be integrated with various analytics and business intelligence applications. HBase would be considered a CP database.

HBase's main features include:
* Linear and modular scalability.
* Strictly consistent reads and writes.
* Automatic and configurable sharding of tables
* Automatic failover support between RegionServers.
* Convenient base classes for backing Hadoop MapReduce jobs with Apache HBase tables.
* Easy to use Java API for client access.
* Block cache and Bloom Filters for real-time queries.
* Query predicate push down via server side Filters
* Thrift gateway and a REST-ful Web service that supports XML, Protobuf, and binary data encoding options
* Extensible jruby-based (JIRB) shell
* Support for exporting metrics/telemetry via the Hadoop metrics subsystem to files or Ganglia; or via JMX

## Document-oriented DB
A document-oriented database, or document store, is designed for storing, retrieving and managing document-oriented information, also known as semi-structured data.

In comparison to the Column-Family Data model it is even richer. A good example system is MongoDB which basically allows you to store JSON objects and the system understands that they are JSON objects so that you can query in ways that take that structure into account.

### JSON
JSON or Javascript Object Notation is a lightweight format for specifying documents similar to something like XML (don't let the name deceive you for it is language independent). An example JSON structure would be the following:

```json
{
  "wines": [
    {
      "type": "Merlot",
      "price": "15.99"
    },
    {
      "type": "Malbec",
      "price": "14.99"
    }
  ]
}
```
The fundamentals of JSON are that they are built on two structures:
* Object: This functions as an unordered set of name/value pairs which are enclosed by {}. The name is a string and the value can be a string, number, true/false/null, or an object or an array (nesting possible)
* Array: An array of ordered list values which are enclosed by [] and are values that are comma separated.

When dealing with structured data there are various optimizations and trade-offs in leveraging JSON in your production servers. Though JSON has many obvious advantages as a data interchange format - it is human readable, well understood, and typically performs well - it also has its issues. Some structured formats, such as Google’s `Protocol Buffers`, are a better choice than JSON for encoding data which you can read about [here](https://developers.google.com/protocol-buffers/docs/overview). Google developed `Protocol Buffers` for use in their internal services. It is a binary encoding format that allows you to specify a schema for your data using a specification language, like so:
```ruby
message Person {
  required int32 = 1;
  required string name = 2;
  optional string email = 3;
}
```
You can package messages within namespaces or declare them at the top level as above. The snippet defines the schema for a Person data type that has three fields: id, name, and email. In addition to naming a field, you can provide a type that will determine how the data is encoded and sent over the wire - above we see an int32 type and a string type. Keywords for validation and structure are also provided (required and optional above), and fields are numbered, which aids in backward compatibility, which I’ll cover in more detail below.

The Protocol Buffers specification is implemented in various languages: Java, C, Go, etc. which means that one spec can be used to transfer data between systems regardless of their implementation language. As you can see, by providing a schema, we now automatically get a class that can be used to encode and decode messages into Protocol Buffer format. `Protocol Buffers` offers very real advantages not only in the ways outlined above, but also typically in terms of speed of encoding and decoding, size of the data on the wire, and more. This is something to look into when building a scalable system. Other example compression types include: `Apache Avro` and `Google Flat Buffers`.

### MongoDB
As it has been discussed above. MongoDB is a document-oriented database that uses JSON-like documents with schemas.

The data model is that the Mongo system in totality holds a set of databases where the database holds a set of collections where the collection holds a set of documents ("table") where a document is a key-value pair that mimics the name-value structure in JSON. What is important to note about the collections is that because they are a set of documents this makes them polymorphic, meaning that they do not need to all have the same schema.

Objects each have a special attributed_id which is unique and identifies an object.

```bash
> x={wine="Merlot"};
{wine="Merlot"}
> db.wines.save(x);
WriteResult({"nInserted": 1})
> db.wines.find();
{"_id": ObjectID("123456789"), "wine": "Merlot"}
```

When we intend to query on such data we use functions like `db.TABLE_NAME.find({'wine':'Malbec'});`.
Other query operators available include `and/or/not`, `in/not in`, some aggregation functions like `count`, `distinct`, and `group`, `sort`, `skip`, and `limit`. In essence, this is a very rich query languague which is unusual for a NoSQL database.

## Overview
Summary of Data Models:
* Key-value: Project Voldemort, Redis
* Generazlied key-value with range queries / limited selection: DynamoDB, GAE datastore
* Column Family: HBase, Cassandra
* Document-Oriented: MongoDB

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

In such a model the actors are processes that are completely isolated, meaning its state is not going to influence any other process. As such, faults can be managed by having a supervisor, that is basically another process, that will be notified when the supervised process crashes and then can do something about it.

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
Database systems with large data sets and high throughput applications can challenge the capacity of a single server. High query rates can exhaust the CPU capacity of the server. Larger data sets exceed the storage capacity of a single machine. Finally, working set sizes larger than the system’s RAM stress the I/O capacity of disk drives.

To address these issues of scales, database systems have two basic approaches: vertical scaling and sharding.

Vertical scaling adds more CPU and storage resources to increase capacity. Scaling by adding capacity has limitations: high performance systems with large numbers of CPUs and large amount of RAM are disproportionately more expensive than smaller systems. Additionally, cloud-based providers may only allow users to provision smaller instances. As a result there is a practical maximum capability for vertical scaling.

Sharding, or horizontal scaling, by contrast, divides the data set and distributes the data over multiple servers, or shards. Each shard is an independent database, and collectively, the shards make up a single logical database.

In NoSQL systems like MongoDB sharding functions this way:
![Sharding-Intro](https://docs.mongodb.com/v3.0/_images/sharded-collection.png)

Sharding addresses the challenge of scaling to support high throughput and large data sets:

Sharding reduces the number of operations each shard handles. Each shard processes fewer operations as the cluster grows. As a result, a cluster can increase capacity and throughput horizontally.
For example, to insert data, the application only needs to access the shard responsible for that record.
Sharding reduces the amount of data that each server needs to store. Each shard stores less data as the cluster grows.
For example, if a database has a 1 terabyte data set, and there are 4 shards, then each shard might hold only 256GB of data. If there are 40 shards, then each shard might hold only 25GB of data.

The specifics of how to shard are unique for each system. For example, in MongoDB a sharded cluster has the following components: shards, query routers and config servers where:
* Shard: responsible for storing the data and providing high availability and data consistency. In a production sharded cluster, each shard is a replica set

* Query Routers, or mongos instances: interface with client applications and direct operations to the appropriate shard or shards. The query router processes and targets operations to shards and then returns results to the clients. A sharded cluster can contain more than one query router to divide the client request load. A client sends requests to one query router. Most sharded clusters have many query routers.

* Config servers: store the cluster’s metadata. This data contains a mapping of the cluster’s data set to the shards. The query router uses this metadata to target operations to specific shards. Production sharded clusters have exactly 3 config servers.

To shard a collection, you need to select a shard key. A shard key is either an indexed field or an indexed compound field that exists in every document in the collection. MongoDB divides the shard key values into chunks and distributes the chunks evenly across the shards. To divide the shard key values into chunks, MongoDB uses either range based partitioning or hash based partitioning.
![Range-Based](https://docs.mongodb.com/v3.0/_images/sharding-range-based.png)

For range-based sharding, MongoDB divides the data set into ranges determined by the shard key values to provide range based partitioning. Consider a numeric shard key: If you visualize a number line that goes from negative infinity to positive infinity, each value of the shard key falls at some point on that line. MongoDB partitions this line into smaller, non-overlapping ranges called chunks where a chunk is range of values from some minimum value to some maximum value.
![Hash-Based](https://docs.mongodb.com/v3.0/_images/sharding-hash-based.png)

For hash based partitioning, MongoDB computes a hash of a field’s value, and then uses these hashes to create chunks.
With hash based partitioning, two documents with “close” shard key values are unlikely to be part of the same chunk. This ensures a more random distribution of a collection in the cluster.

These types of sharding techniques are optimized for certain queries. This in essence is similar in intuition to hash indexes and range indexes. Range based partitioning supports more efficient range queries. Given a range query on the shard key, the query router can easily determine which chunks overlap that range and route the query to only those shards that contain these chunks. However, range based partitioning can result in an uneven distribution of data, which may negate some of the benefits of sharding. Hash based partitioning, by contrast, ensures an even distribution of data at the expense of efficient range queries.

### Challenges in Multi-Node Systems
So what are some challenges that exist when we move towards distributed systems.

* Processing queries across multiple sites: i.e. What if I am joining two tables that are at different sites
* Running distributed ACID transactions: i.e. How do we guarantee isolation such that if one of the transaction participants crashes in the middle
* Keeping replicas in sync: i.e. What level of consistency do we wish to maintain on the replicas?
* Dealing with failures where nodes goes down (i.e. after a replica is created, we need to take care of failure handling) or the network is partitioned (i.e. Even though all the data is still there, other node's might not and could get worried)

We will now be doing a small case-study into distributed queries and different join strategies.

#### Distributed Queries
Our example use case will be querying on wines:

```sql
SELECT AVG(W.age)
FROM Wines W
WHERE W.rating  > 3
AND W.rating < 7
```

Let us say that the data is horizontally partitioned. Where the wine tuples with rating <5 are in London and the wine tuples with a rating of >=5 are in NYC. As such, to calculate the `AVG` we would need to compute the `SUM` and `COUNT` at both sites while for the WHERE clause, tuples contained with W.rating>6, would use just one site.

Let us say that the data is vertically partitioned. Where the wine tuples would have the `wid` and `rating` at
London, `name` and `age` at NYC, `jid` at both. `wid` would be the wine tuple `id` while the `jid` would be some join id. As such, you could reconstruct the full relation by joining on `jid`, then evaluate the query. This type of system would be used if locally only some selection queries were needed per geographical region and the full relation wasn't necessary that often.

#### Distributed Joins
* Strategy 1: Compute and fetch as needed (If query was not submitted at NYC, must add cost of shipping result to query site)
* Strategy 2: Send to one site and evaluate join locally (however, if the result size is very large, may be better to ship both relations to result site and then join them)
* Strategy 3: `Semi-join.` In this case, we are trying to avoid shipping all of the NYC to London, for why can't we just ship the tuples that we will join with i.e. London values are projected onto join columns and shipped to NYC where they are joined with NYC specific values, giving the necessary reduction desired, and eventually shipping back to London to be joined with the full expansion of London's values.

**Bloom Filters**

Last strategy is leveraging Bloom Filters. This approach is similar to semi-join, but avoids shipping
the whole projection as the result is expressed as a smaller bit-vector instead. This data structure is called a Bloom filter. For example, let us assume that you have some relation that resides at London. You would use relation to compute a bit-vector of some size k where you would hash the join attribute values into range `0 --> k-1`, where if some tuple hashes to I, set bit I to 1 (I from 0 to k-1); this is the bloom filter. We would then ship this bit-vector to NYC. At NYC, you would hash each tuple in NYC similarly, and discard tuples that hash to 0 in the London bit-vector which gives us the reduction of R with regards to S as desired. You would then ship this reduction back to London, where the join will take place. The advantage in such a strategy is that the bit-vector is cheaper to ship. This type of system obviously comes with some drawbacks: false positives are possible i.e. if the Bloom Filter says an element is NOT in the set, this is definitely true, but if the Bloom Filter says an element IS in the set, this may or may not be true.

## Hadoop MapReduce and Apache Spark
Why build all of these systems in a distributed nature? What advantages can we leverage with our system setup this way for data analytics.

In comes `Hadoop MapReduce`. `Hadoop MapReduce` is a software framework for easily writing applications which process vast amounts of data (multi-terabyte data-sets) in-parallel on large clusters (thousands of nodes) of commodity hardware in a reliable, fault-tolerant manner.

A MapReduce job usually splits the input data-set into independent chunks which are processed by the map tasks in a completely parallel manner. The framework sorts the outputs of the maps, which are then input to the reduce tasks. Typically both the input and the output of the job are stored in a file-system. The framework takes care of scheduling tasks, monitoring them and re-executes the failed tasks.

Typically the compute nodes and the storage nodes are the same, that is, the MapReduce framework and the Hadoop Distributed File System ([HDFS Architecture Guide](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)) are running on the same set of nodes. This configuration allows the framework to effectively schedule tasks on the nodes where data is already present, resulting in very high aggregate bandwidth across the cluster.

The MapReduce framework consists of a single master ResourceManager, one worker NodeManager per cluster-node, and MRAppMaster per application ([YARN Architecture Guide](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)).

The format of MapReduce is understood best with the canonical WordCount example:

![Word-Count](https://cs.calvin.edu/courses/cs/374/exercises/12/lab/MapReduceWordCount.png)

So what is MapReduce? It is a part of the Hadoop framework that is responsible for processing large data sets with a parallel and distributed algorithm on a cluster. As the name suggests, the MapReduce algorithm contains two important tasks: Map and Reduce. Map takes a set of data and converts it into another set of data, where individual elements are broken down into tuples (key/value pairs). On the other hand, Reduce takes the output from a map as an input and combines the data tuples into smaller set of tuples. In MapReduce, the data is distributed over the cluster and processed.

The difference in `Apache Spark` is that it performs in-memory processing of data. This in-memory processing is a faster process as there is no time spent in moving the data/processes in and out of the disk, whereas MapReduce requires a lot of time to perform these input/output operations thereby increasing latency. You can read more about Apache Spark [here](https://databricks.com/spark/about) but here is a general breakdown of what Spark is and what advantages it provides in distributed data analysis.

Apache Spark provides programmers with an application programming interface centered on a data structure called the resilient distributed dataset (RDD), a read-only multiset of data items distributed over a cluster of machines, that is maintained in a fault-tolerant way. It was developed in response to limitations in the MapReduce cluster computing paradigm, which forces a particular linear dataflow structure on distributed programs: MapReduce programs read input data from disk, map a function across the data, reduce the results of the map, and store reduction results on disk. Spark's RDDs function as a working set for distributed programs that offers a (deliberately) restricted form of distributed shared memory that is also able to store in-memory using different types of distribute in-memory storage solutions like `Alluxio`. `Alluxio`, formerly `Tachyon`, provides Spark with a reliable data sharing layer, enabling Spark to excel at performing application logic while Alluxio handles storage

The availability of RDDs facilitates the implementation of both iterative algorithms, that visit their dataset multiple times in a loop, and interactive/exploratory data analysis, i.e., the repeated database-style querying of data. The latency of such applications (compared to a MapReduce implementation, as was common in Apache Hadoop stacks) may be reduced by several orders of magnitude. Among the class of iterative algorithms are the training algorithms for machine learning systems, which formed the initial impetus for developing Apache Spark.

Apache Spark requires a cluster manager and a distributed storage system. For cluster management, Spark supports standalone (native Spark cluster), Hadoop YARN, Apache Mesos, and soon Google Kubernetes. For distributed storage, Spark can interface with a wide variety, including Hadoop Distributed File System (HDFS), MapR File System (MapR-FS), Cassandra, OpenStack Swift, Amazon S3, Kudu, or a custom solution can be implemented. Spark also supports a pseudo-distributed local mode, usually used only for development or testing purposes, where distributed storage is not required and the local file system can be used instead; in such a scenario, Spark is run on a single machine with one executor per CPU core.
