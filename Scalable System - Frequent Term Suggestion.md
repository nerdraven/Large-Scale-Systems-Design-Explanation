## What "Top Query Suggestion System" looks like?

This system helps companies to use and integrate typehead system with only the web service. This web service provides companies to suggest
top queries to companies users when they write something in the search box. The system basically occurs in two parts.
First one is related to the registration system. It is like a web system. In this part, companies can register this system and system do 
following steps for the related company. If company register the system, the first system gives web service to a registered company to 
use and integrate it. The main approach and aim for this system are saving the companies from typehead (query suggestion) system. 
They can use suggestion system just to register this system. If they register, the system automatically reserves servers, databases and 
storages. This means that, when companies register the system, they don't need to deal with the servers, databases and storages. 
Just registering the system is enough for use it. Additionally, In using process, registered companies just pay a fee as they use the 
system. (Depends on the database storage and servers). The next goal of us is giving chance to companies for selecting a number of 
servers and databases (for sharding and replications). Moreover, the second part of the system is basically web service. If companies 
register the system, the system gives "top query suggestion system web service" to registered companies. They just call the web service 
and use it. When web service is called system will automatically execute the process in reserving servers and databases for companies. 
Notice that, technical details and system properties
will explain parts.

## Requirements

There are three types of requirements.

### Functional Requirements

Companies can register the system.
When companies register the system, registered companies can use the system with giving web service.
Companies can cancel their registration.
When companies can cancel their registration, their servers and databases should be adjusted.
When companies integrate this web service to their system, the system should start to suggest queries to users when writing letters in 
    the search box.
The system will start to suggest queries if the user has not pressed any key within 50 ms.
Before starting writing, the system should wait for getting the first letter.
The system should save the recent history based on the user. Notice that, these records can be used for suggestions to any other users.
A connection should be established as soon as possible. The best thing is that when a user opens the web server, a system is ready to use.
The system should suggest between 0 and 10 terms at down the box. This means that system should suggest the top 10 terms.

***** NEXT THOUGHTS WANT TO BE DEVELOPED-------

Companies should choose the number of the databases and application servers.
Companies should pay a fee to use the system.
Companies should determine suggestion process. They should be able to choose the properties that they want to prioritize.
A system should have one more layer that works for eliminating unwanted words. (Depends on the companies.) To illustrate this, they can 
    choose the most searched queries, the newest queries, the language of the queries, the most clicked
properties, the location of the queries that searched before and etc.. (This features will be determined later.)
Companies should choose the suggestion query number. The default value is 0 between 10.
Notice that, the server can push some part of their cache to CDNs for providing efficiency.

### NonFunctional Requirements

* The system should be highly available. This means that, if we think that system should be usable at every time, the system should be work 
nonstop.
* The system should have the real time. This means that system should be a real-time system. Another possible restriction is that system 
should suggest within 50 ms.

### Extended Requirements

The system should keep the top queries because of giving the top queries.
Notice that if use goes to any website via suggestion query, this query classified as an important query.
Additionally, the system should be monitored. This means that, if any company has a huge data and needs to be one more shard, then 
companies should automatically be alerted. This alerting system will be active when database usage reaches the %80 and application 
response time increase the over 100 ms.

## Capacity and Estimation

Let's imagine that if the system takes the 1 billion searches every day from total companies, which gives us to approximately 10 K 
queries per second. We can eliminate duplicate words and they just affect us top finding top terms. But we can assume that only 1:5 of 
these will be unique. Let's assume that we will have 10 million uniques terms for which we want to build the index.
if every query has an average of the 3 words and lengths are average 5 characters. This means that query length is 15 characters 
(average) So each query size is average 30 bytes.
=> Daily = 10 M * 30 Byte = 300 MB
=> One year => 300 MB + (365 * 3GB * 0.02) = (approximately 20 GM) (For coming new queries we assume %2)

## Technical Details

Our system is occurred two parts. First one is related to registration second one is algorithm and web service that will be presented.
First, part will be developed basically with the Java, Spring, Hibernate, MySql (ORM and AOP). The second one will be developed 
basically with Java and MongoDB.

```bash
Other techniques that will be used is;
- Amazon EC2
- Amazon S3
- Amazon SQS
- Replication
- Sharding
- Load balancer
- Amazon ElastiCache
- Amazon cloud watch
```

## API's

We can use SOAP or REST services, but I prefer that we use
There are mainly two types of API for web service. First one is basically related to suggest queries. The second one is related to 
sending a searching query to the system. Most important parameter of API's is applicationID. ApplicationID is given companies when they 
register the system.

## Database System

We will use two types of databases. First one is RDMS (MySQL), second one is NoSQL (MongoDB). MySQL is used to save companies and their 
information, NoSQL is to save queries. Additionally, replication and sharding mechanism will be explained below parts.

## Design and Algorithm

* Notice that this system has more complex problems. To illustrate this, if our database save some terms like a "mountain", "money", 
"monkey" and "movement" and user write the word such as "mo" then system should suggest all of this words ("mountain", "money", "monkey",
"movement"). Because of the giving lots of queries with minimum latency, we have to efficiently store the data that can be queried 
quickly.

* For this operation, we need to store our index in memory in a highly efficient data structure.
To handle this problem, the best option is using "Trie". Trie basically provides us with word letter by letter.(like as a tree)
For processing, the query starts with "Mo" system should traverse all the words start with "Mo". Notice that if any trie node has exactly
one child node. Then we can combine these nodes.
Notice that if we do this system with case insensitive, then we can store the data all capital letters. So there is no any problem 
related to case sensitive.

* Another complex situation is finding the top queries. The first solution is keeping the searched count of all words (in the last letter 
of every word.) But this gives much more problem. Imagine that system is so huge and to find the top words, we have to traverse all the 
words. This causes to system performance decreasing. We should improve our solution to find best efficiency. We can store the top words 
in each node that starts with this node. This provides to increasing performance and speed up. But more improvements, we can store the 
referential point and just traverse these points for suggestions to handle the more memory usage.
We built the tree with the bottom up principle because of the keeping top words. (we recursively call all child nodes to calculate
top words count.)

* Another problem is updating trie. For example system has to give 100K queries in a second. This means that we should not update trie 
after every query processing. We can have a Map-Reduce process, and every periodical times (are determined by us). Trie is updated with 
Map Reduce. Notice that we should do that when system offline because system should'nt block the trie. If we have a master-slave 
configuration and having one more tries, the system works perfectly. This provides to better performance and being availabity and 
having minimum latency.

* If we don't suggest any word because of legacy or piracy, we can add a filter to each server and we do not show these words.
One more think is that we should take care of the user location, language and personal history to suggest any words to providing good
matching.

* Another think is that we can take snapshot of our trie periodically (is determined by us). So we can easily rebuild the trie when server
goes down. We can store the trie starting with the root and continuning to saving level by level.

## Data Partition

Even if our index is fit to keep one server, we can do data partition to get higher efficiency and lower latencies.
There are three option for doing partitioning. First one is a range based partition, the second one is partition based on the maximum 
capacity of server and third one is partition based on the hash of the term. The first one is not efficient because when we save the data
based on the first letter, some words can be much more than the other if we compare words related to starting the letter. For example, 
if we save all the terms that starting with the 'A' in one server and save all terms starting with the 'B' another server, some servers 
can save much more data then others. Another problem is that maybe, words starting with 'A' cannot be fit on one server. So second 
option more useful than first one. We store data on a server as long as it has memory available.

*Server 1, A-AAD
*Server 2, AADA, CDA
*Server 3, CDB - EKZ
---

If the user writes 'A' in the box, both one and second sever have to use, but if the user writes AB only server 2 has to use. Notice 
that we can use the load balancer for determining the suitable server. Additionally, if we use one more server to suggest, we need to 
merge results on the server side or client side. If we prefer to do this on the server side, we have to have another layer between load 
balancers and trie servers. It provides a system to return top suggestions. If there are lots of queries for terms starting with 'mo', 
server keeping it will have a high load compared to others. So third option is best for partitioning. Each term is passed to a hash 
function and then is stored to the server via this hash function. (we have to use consistent hashing for fault tolerance and load 
distribution)

## Replication and Load Balancer

We should have replicas of our trie servers both for load balancing and fault tolerance.
Notice that if one server goes down, thanks to the master-slave configuration, the slave can take over and it becomes a master, server 
that went down can rebuild the related trie servers based on the last snapshot.

## Caching

We can cache the top results and if we do that, we get much more good performance. We can have separate cache servers in front of the 
trie servers. Firstly application server should check before going to trie servers and if application server finds, it returns these 
queries without going trie servers. But we should build caching mechanism to hold top queries. That's because top queries can change. 
We can add control time for providing consistent caching.

## Getting Better Result

To get a better result, users will encounter with some suggestions based on the location, previous searches, languages and etc. 
We can store personal history on a different server (trie servers) and cache them.

## Monitoring

As we said before, the system should monitor based on the usage of the database server and query response time for companies.
