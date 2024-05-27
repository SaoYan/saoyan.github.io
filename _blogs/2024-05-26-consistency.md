---
title: "What the hell is consistency?"
author: yiqi
permalink: /blogs/2024-05-26-consistency
date: 2024-05-26
collection: blogs
tag:
- Distributed system
---

- [🤯](#)
- [In the world of data/system modeling](#in-the-world-of-datasystem-modeling)
- [In the world of data replication and CAP](#in-the-world-of-data-replication-and-cap)
  - [Data replication consistency](#data-replication-consistency)
  - [Understanding CAP theorem](#understanding-cap-theorem)
- [In the world of transaction](#in-the-world-of-transaction)
- [Consistency levels and Isolation levels](#consistency-levels-and-isolation-levels)


# 🤯

Whenever we read any material on distributed systems, or system design articles, the word "consistency" flies everywhere and quickly confuses our mind. It seems to always mean the same thing: "the system being in a good state", yet they feel so differently sometimes, especially when mixed with other terms such as CAP, linearizability, serializability, and isolation/consistency levels.  

In this article, I'll try to come up with a practical mindset. Hopefully it will help you, not make it worse.  

So let's get started.

# In the world of data/system modeling  

In most contexts of discussing high-level data models and/or system architecture, the word "consistency" does NOT involve any technical concepts. It is as simple as "the system should be in good and correct state".

Example 1: When designing the entity model of a database, you may want to put some constraints, with the purpose of "making sure of data consistency". 

Example 2: When designing a digital wallet, one of the most basic requirements can be described as "consistency". To be more specific, on money transfer A->B, the sum of A and B's balance should not change. From the technical perspective, this is an atomicity requirement. When describing the high-level design, people do use "consistency" quite often.  

# In the world of data replication and CAP  

## Data replication consistency  

When talking about data replication and CAP, the following terms are equivalent:

* consistency  
* strong consistency  
* linearizability  

Definition: As soon as any client successfully completes a write, any other clients must be able to see the latest value just written.  

Another related term is "eventual consistency", which is basically "NOT strong consistency". Data will eventually be replicated to all nodes asynchronously, and some clients can read outdated values.  

## Understanding CAP theorem  

Following the above definitions, the CAP theorem can be described as:  When some nodes disconnect with others due to network partition(P), an application need to trade off between availability(A) and consistency(C):  

* If we choose high availability, the application must continue to serve requests, with the consequence that reading from some nodes will return outdated data.
* If we choose strong consistency, the application has to reject write requests because some replica nodes are unreachable, resulting in impacted availability. 

Learning by case study is always helpful, so let's think of a real example: A leaderless database with quorum  

* Happy case: W + R > N guarantees strong consistency.  
* On network partition, less than W nodes are reachable during write. We have 2 options:
  * High availability: Adopt "sloppy quorum", which breaks quorum requirements and leads to non-consistent read.  
  * Strong consistency: Reject write requests, making the application unavailable.  

# In the world of transaction  

"Consistency" is the letter "C" in the ACID properties of database transactions. It has the same meaning as the consistency in "data/system modeling", i.e. "system being in a good state". Specifically, it usually promises that transaction results follow some data constraints defined from the business perspective.

It is worth mentioning that "isolation" (the letter "I" in ACID) is also known as "serializability", which can be easily confused with "linearizability". Serializability requires that when multiple transactions execute concurrently, the final state of the system must be as if all those transactions had executed one by one (in "serial").  

# Consistency levels and Isolation levels  

In most database documentation, "consistency levels" sections usually refer to the consistency of data replication, and "isolation levels" are about transaction isolation (serializability).  

Example:
* [Cosmos DB Consistency Levels](https://learn.microsoft.com/en-us/azure/cosmos-db/consistency-levels)  
* [PostgreSQL Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)  
