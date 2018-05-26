## General Overview

If we decided to develop a large scalable system like facebook messenger design, firstly we should design a system carefully. A lot of 
architects say that not only writing code carefully but also designing architect is so much important to provide maintenance and 
reliable system.  As we mentioned above if we decided to design messenger system like Facebook Messenger System or Instagram Messenger 
System, we should think all design parts carefully. One more thing is that we have to design our system in a structure that can run 
smoothly for 5 years. Although initially, the system is not under heavy traffic, applications grow exponentially and it is very 
difficult to make changes afterwards. So let's start to design Facebook Messenger System.

## What is Facebook Messenger?

Facebook Messenger is a software application that provides text-based message service in real time. Real-time means that system should send messages and clients should receive these messages in real-time. In order to design real-time system, a database should be designed carefully or partition operation should be done very very carefully. So briefly, Facebook Messenger is a text-based messaging service in real-time. You can sen text-based message to their Facebook friends. Notice that this system need to work on both platforms, e.g phones, and websites.

## Requirements

This section is crucial for determining the requirements of the system. For example, user stories can be roughly defined in this section.
There are three main steps to define all requirements which are functional, non-functional and extended requirements.

### Functional Requirements

* System has to support one-to-one conversation.
* System has to support group conversation.
* System has to keep all message histories.
* System has to support to send messages to online users.
* System has to support to send messages to offline users.

### Non-functional Requirements

* System should be highly available. Users should be able to use the system whenever they want.
* System should work with real-time experience.
* System should be highly reliable. This means, no messages should be lost and each message must reach the recipient.
* Consistency is very important in our system. Users should be able to see same messages on every device.

### Extended Requirements

* As we mentioned above, system should be able to support group chats. We need that database should be designed with this context in mind.
* If we talk about the messaging system like Facebook, Twitter or Instagram messaging service, push notifications are so much important. 
* If we integrate notifications into the system, users do not need to check whether new messages come in. We can integrate AWS SNS, 
Azure Notification Hub or OneSignal to handle this problem.

### Estimations

It is extremely important to estimate capacities in the design phase of large systems. for example, if we do the capacity building 
correctly, we can find out how many servers we need after 5 years. This helps us to define sharding procedure. For example, we 
initially shard the system on a single server. In the ongoing process, we can scale easily when new servers are added to the system. 
So let's start estimations.

* Number of Total User: 500 million
* Number of Daily Active User: 50 million and each send 10 message in every day.
* Number of Messages Sent Per Day: 500 million
* Let's estimate each message 100 Byte. Total daily message capacity is 500 million * 100 byte = 5*10^10 byte = 50 GB.
* Let's think 5 years later. 50 GB * 30 * 12 * 5 = 900 TB (Just only message data)
We need to take care of metadata. Metadata keeps the user information or message information likes userID, messageId, location, sending 
date.

Another important point to estimate capacity is thinking replication. We have to consider the replication count and data capacity. 
They require 3 or more replicas on ideal scalable systems.One more point is the entire capacity of each server should not be used. 
The best case scenario is that the maximum 80 percent of the capacity is used and the remainder is not used. (80-20 rule)
So let's talk about bandwidth estimation. What we need to be aware of here is that we must have the same bandwidth while sending and 
receiving the message. Bandwidth is the amount of data that goes out in seconds. Assuming that 50 GB of data per day is sent, 
approximately 6 MB of data is sent at the moment. Both of upload and download have to same bandwidth capacity. So total bandwidth 
capacity is approximately 12 MB.

The final issue in estimation is the cache estimate. Every user wants to see easily the data that they have sent. On the system, 
we can keep the last 3 days of data for each user in the cache. Maybe we can handle the cache problem with using LRU approach. 
Not every user uses the system at the same frequency. We can give more importance to hot users by using LRU approach.

## High-Level System Design

To put it in simple terms, suppose that a user sends a message and the other user needs to receive the message. The flow diagram of the 
message should be as follows.

* UserA sends MessageA.
* MessageA goes to Chat Server.
* Chat Server connects with Data Storage before sending messageA to UserB.
* Chat Server sends MessageB to User2.
* How does the UserA know that the message goes to the server successfully? After sending a message, a message goes to a server in packages.
* The server sends a message to the user when the package is received. This is acknowledgment. There can be some special problems when a 
message sent. Notice that we can use two different servers to design this architecture. We will explain in detail later.

## Detail Component Design

As we mentioned above, Basically if we're talking about a single server, we need to handle following problems.

Receive and deliver messages sequence. When UserA sends the message and then sends it after B in B then it has to go before B then before
Store and retrieve data from DB quickly and efficiently.
Offline users should be considered.
If a message is sent to a user offline, the message must be saved one time and sent when online.
How can send message efficiently?

There are basically two models we face which are Pull model and Push mode.

* Pull model: Users periodically ask to a server at the push model. The main problem with pull model is this is not a real-time operation. 
If we would like to catch real-time operation, clients should be able to ask frequently. This means server may send empty response a 
lot of time.

* Push model: There is an open connection and send when notified. It brings this system closer to real-time. The push model is a briefly 
open connection and, when the server receives a new message, it will send a message to receiver immediately. Examples of push model are 
socket and long pulling. In the long polling, the server doesn't send a response before coming new message. This means the server 
doesn't send an empty message. It helps system to reduce resource usage. Notice that, in the long-polling, a connection will be closed 
after a message is sent. We can face connection problems such as timeout or disconnection. In such cases, the connection must be 
re-established. Another option is web socket. A web socket is truly real-time and this is the best option to design messaging systems. 
Long-polling is also a real-time operation but web socket is a little bit faster.
How can server keep track of all open connection to efficiently redirect messages to the user?

* The server can use a hashtable to keep track of all open connection to efficiently redirect messages to a relevant user. Hashtable is a 
key-value pair that keeps the userID and connection object. When a new message comes, the server looks the hashtable and find connection 
object.

How can we handle offline users problems?

Firstly, we need to store messages. As soon as the user is online, the client asks a server there is an incoming message to retry them.

How many chat servers do we need?

Let's say 50 million connection at the same time. Assume 50K connection at the same for each server we need to 1K server to reduce 
traffic heavy.

How to use servers efficiently?

Our scalable system has a lot of servers. and when a new message comes, a system should need to know which server should be used. 
To handle this problem we are using a load balancer. A load balancer is mapped by using UserID. If we try to use messageID, a system 
is far from real-time. Additionally, as we will mention later, we need to manage all replicas with intelligent round robin approach.

Sequencing?

The order in which messages arrive in messaging systems is very important. Here we can use a timestamp for solving the sequencing 
problem but if we use a timestamp, we cannot ensure as there may be connection failure. Another approach is sequence number.  
We add sequence numbers to each message. The system does not show the one after taking the previous one.

Store and retrieve a message from the database efficiently

We have multiple options to store data quickly. The first one is to start a separate thread. It just works for store data in the 
database. Another option is to send asynchronous request to store into database. We can handle following problems when data is stored.

How to efficiently work with database connection pool?
How to retry failed request?
How to storage system should use efficiently?

In systems such as messaging, insert operation is more from the update operation so we need to choose the database that is fast for 
insert and read and slow for updates. We need to fetch data quickly. There is a huge number of small messages comes and insert.

We cannot use RDBMS like MySQL or NoSQL like MongoDB. Every time reading and writing operations in such databases are not efficient. 
We can use wide column database and this is the best option for us such as HBase.

How should clients fetch data from the server efficiently?

The number of data may vary depending on the device used such as cell phone and web. If we fetch data with pagination we can handle 
the efficiency problem.

Managing User's Status 

We need to keep track of all user status. When a status change occurs, all relevant user should be notified. A lot of changes can be 
done and if the system tries to notify all relevant users, a lot of resources may consume.

When the client starts the application, pull starts to friend list.
When a message is sent an offline user, the system throws failure and updates status.
Notice that each message means new pull operation. Pull operation can be done at regular intervals.

Notice that each of chat server connects with each of DB Shards with the load balancer.

## Data Partition

There is a lot of data we need to partition so we need to use sharding to solve the problem. This is called scale-out approach. 
By using a load balancer we make the use of shards effective. Server load balancer can understand which shard to go to. 
The important question here is that by which parameter should we determine the shark shards? Each user wants to see their own messages 
quickly so we want to keep their messages on the same server. This may not be a uniform distribution, but it is an effective way for a 
real-time experience.

Let's assume each DB Shard is 4TB. So we estimate that five years later our data capacity will reach 900 TB, so the total number of 
shards required is 250.

Our function is userID % 250

At the beginning of the design, a single server will suffice. However, multiple sharding on a single server will make our next steps 
easier when the system needs to have multiple sharding. One more thing is partitioning by message ID will slow down the system.

## Caching Mechanism

To quickly show users' messages, we can keep some messages in memory. The amount of this depends on us. For example, we can keep your 
last 3-day message or the last 15 messages.

## Load Balancer

The load balancer is intended to relieve and balance the load on the servers as is known. The load balancer can be located on the system 
such as;

* In front of chat servers
* In front of database servers (DB Shards)
* In front of cache DBs.

## Fault Tolerance

Mistakes on systems such as messaging should be compensated without notice to users. The best way to handle server die problem is 
replication. Replication is multiple copies of servers. If you want to destroy a server in the system, then what should be done? 
The Reed-Solomon approach is an appropriate solution to bring back lost data.

## Extended Requirements

* Push notification: This should be used for offline users. AWS SNS or Azure Notification hubs are usable for this feature. 
* Group chat:
* Monitoring: We need to monitor the system to see if the system is in any trouble or if we need a new server.
