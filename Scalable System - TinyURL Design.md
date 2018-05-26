## Why we use URL Shortening Service?

This application helps us to save lots of space. To illustrate this, if we send mail or push notification for mobile phones, 
then we can save a lot of space. In addition, hiding the actual URL is provided with this service. Notice that shortener URL service 
is nearly 1/3 size of actual size.

## Requirements and Goals of System

There are many important requirements for URL shortener service. When the user gives the actual URL, this actual URL should save 
database and return the short URL to the user. Notice that short URL’s should be unique. When a user can enter the short URL, 
the system should redirect the user to actual URL. Notice that this short URL should have an expire time. Moreover, some performance 
of the system should be provided.

The system should be highly reliable. This means that every actual URL should be saved and not deleted.

The system should have a minimum latency. This means that when a user enters the short URL, the user should quickly redirect the actual 
URL with minimum latency. For this purpose, we use data sharding and caching.

The system should be highly available. This means that when the user enters the short URL, he or she should reach the actual URL at the 
same time. For this purpose, we can use data replication. The user can save the short URL with a custom alias. But for this property, 
we can restrict user with character limits.To monitor system, we can save the access time, access region, browser or platform. 
Finally, these short URL’s should not be predictable.

## Capacity Estimation

Let’s think about our system has 100 million users and each user enters the five new URL in a month. These means that every month, 
500 million new URL comes.If we think about every actual URL size is 100 bytes (100 characters represented)  then in a month, 
100 bytes * 500 M = 50 GB new data comes. This means that in a 10 years later, 50 GB * 12 * 10 = 120 * 50 GB = 6000 GB = approximately 
6 TB data must saved. Notice that if the ratio of read and write operation is 10: 1 then 500 million new URLs writes and 5 billion 
URLs reads.

### Traffic estimates

In a second = 500 million / (30*24*3600) = ~200 URL / s to write
In a second = ~200 * 10 = 2K URL / s to read

### Bandwith estimates

In a second = 200 URL * 100 bytes = 20 KB / s to write
In a second = 2 K URL * 100 bytes = 200 KB / s to read

### Caching estimate

Notice that if we realize that %20 rule of memory.

In a day = 2 K * 3600 * 24 * 0.2 = ~35 GB

## System API’s

We can use REST API to connect to system and devices. There are two main APIs that are createURL and deleteURL. All of these have 
develop_api_key for controlling the usage and limit of using API.

*createURL - returns string short URL
*deleteURL - returns HTTP API response.
*controlExpireTime return HTTP API response.

## Database Design

For this software, we can use NoSql like as DynamoDB, MongoDB or Cassandra. Amazon provides Dynamo databases for this purpose. Why we 
can use NoSql is easy. We have lots of URLs and any URL has no relation to other URLs. We basically have three tables. One is URL table, 
second is User table and third is URL and User relation table. NoSql has no relationship between tables and NoSQL has no support ACID 
properties. This basically works with CAP Theorem.

## Basic System Design

We think that how can we provide actual unique short URL and how can we format actual URL. The simplest way to encode URL by using MD5 
or SHA256 hashing with base36 or base62 or base64 hashing. So we think that 8 unique letters to use hashing and base64 is used, then 
approximately 280 trillion possible strings are created and this number is big enough for unique URLs.

Think basic:

*1. Client enter the actual URL
*2. Short URL goes to the server
*3. Server encode the URL and try to save database
*4. If the database has this encode URL return fail and want to new short URL to the server
*5. If the database has no this short URL and it saved database and server returns short URL. But how we can handle unique key operations.
For this purpose, we think that user can be online or offline. So we can use key generation service for this purpose. Key generation 
service has two databases and one for keeping not used the key and another for keeping the used key. Notice that key generation service 
created unique keys before and this provides minimizing latency. If we have one more server then we must synchronize the key to eliminate
obtaining the same key on concurrent time.

## Data Partitioning and Replication

Billions of URL should be saved and because of that, we need to the data partition. We can prefer two types of partition. First one is 
the horizontal partition and the second one is the vertical partition. For this software, we use a horizontal partition. We can use 
hash-based partitioning. With this property, our hashing randomly distribute URLs into different partitions. For replication, we can 
replicate all servers at least 2 another copy services.

## Cache

We can use Amazon elastiCache for caching operations. LRU caching mechanism uses for this purpose. Notice that cache data is a mirror 
copy of actual data. Each copy data should have an expiry date.

## Load Balancer

* Each server layers can have the load balancer.
* Clients and application servers.
* Application servers and database servers.
* Application servers and cache servers.

We can use round robin but this has some problems so we can use intelligent LB approach.

## More Visually

Clients enter the URL. Proxy server takes the requests. The proxy server looks the request whether its cache has this short URL or not. 
If not then redirect the request to the suitable application server by the load balancer. If the request is related new URL, application 
servers contact with Key generation service.(key-DB) Application server redirects the request cache servers and database servers by 
load balancers.
