---
layout: post
title: "NoSQL Databases"
---

# (1 Week)

# Introduction
In this section of the course we will be doing a survey into data storage beyond the relational model and RDBMS.

* "NoSQL" systems (Redis, MongoDB, Couchbase,MongoDB)
* New processing models (MapReduce, BSP/GraphLab…)

Starting in mid-2000s, custom data storage solutions developed in industry that include and are not limited to: Amazon DynamoDB, Google Bigtable, Facebook Cassandra, etc. Now these solutions are available for general use in various ports/variants. 

Sometimes an RDBMS is more than we need as we do not really care so much about expressive queries or
transactional guarantees. An example application would include: processing internet logs where  append/retrieve operations are essential but 
complex queries, transactions, etc are less so. Other reasons to move to noSQL are for horizontal scalability. As cheap commodity machines are becoming more and more common, the need to scale horizontally instead of vertically (where one needs to buy bigger, and fancier servers) is seemingly the most economical. 

Other problems that you are likely to encounter is the issue for when data has lots of NULL values. Data may not be as complete as the relational model that is forced to fit under. As such there could be a case where there are many "missing" (optional) attributes. This would mean that there would be NULLs all over the place which would be quite painstacking considering the complex ISA heirarchy that you established when building out the relational model. Here is an example schema: 

|          Wine         	|       Region       	| Parker Score 	|
|:---------------------:	|:------------------:	|--------------	|
|       Pinot Noir      	| Williamette Valley 	|      90       |
|         Merlot        	|   Saint-Emillion   	|      95       |
| "Ilans Tutti Fruitti" 	|   Ilan's Backyard  	|     NULL      |

In such a system if Ilan's Backyard were to make a bunch of wines, the chances of Richard Parker immediatly giving a score is slim. As such, there would be MANY missing values in this type of relational model. 


The other issue is that relational schemas are hard to change, especially if you frequently add new fields to your data. Let's say Ilan's Backyard would like to add "style of music" to express the optimal music that a wine should be paired best with. The addition of this new field would be quite expensive to add to the entire relational model, despite my eagerness to add more information towards explaining my wine:

|          Wine         	|       Region       	| Parker Score 	| Music Pairing 	|
|:---------------------:	|:------------------:	|--------------	|---------------	|
|       Pinot Noir      	| Williamette Valley 	| 90           	| NULL          	|
|         Merlot        	|   Saint-Emillion   	| 95           	| NULL          	|
| "Ilans Tutti Fruitti" 	|   Ilan's Backyard  	| NULL         	| Swing         	|
| "Ilans Deep Red"      	| Ilan's Backyard    	| NULL         	| Bachata       	|

This is where our exploration begins. What does data look like if it is not relational? 
* More complex and/or flexible thatn relations (Trees / Nested Document Structure)
* Simpler than relations (Key-Value pair)
* LOTS (you are encouraged to explore!)

The field of NoSQL covers all of the above. 

We will not take a deep dive with the topics below:


## Data Storage Solutions
### Key Value Stores
#### Project Voldemort
#### Redis
#### DynamoDB

Secondary Indexes

CAP Theorem

ACID

Consistent Hashing

Replication

Data Versioning and Quorums

#### Google App Engine Datastore

### Column Family data model
#### Apache Cassandra
#### Apache HBase

## Document-oriented DB
### MongoDB

Data model

Querying

## Map Reduce

## Distributed Systems 
### Sharding
### Transactions
### Bloom Filters
### Failure Detection 
