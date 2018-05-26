### WHAT IS PASTEBIN

* This website helps users to upload and store text plain or image in cloud platforms. Users can easily reach this plain text or image 
with given unique URL. Additionally, other users can easily reach plaint text or image shared by other users by accessing URL. 
Additionally, users can share plain text or image with private or public options.

### SYSTEM REQUIREMENTS AND GOALS OF SYSTEM

* Users should upload and store plain text and image then take a unique URL. Users should reach their uploaded plain text and image by 
entering this given unique URL. Users should choose public or private properties to determine which users can see files or which users 
cannot see files. Other users should see the public files by accessing specific unique URL. Notice that system should be highly reliable. 
This means that every files should store and when server fail, files should not be lost. (Replication)
Notice that system should be highly available. This means that users should access the website whenever they want. The system should 
return the response with minimum latency. This property is provided by using load balancing, partitioning, sharding, caching. 
Additionally, the system can be monitored.
Rest or SOAP API's can prefer for the connecting system and outer environment. We can limit users to determine to upload maximum size 
file. (such as 5 MB)

### Capacity Estimation

* Let's assume that in one day 1 Million new file comes and the ratio of read-write operation is 5:1

### Traffic estimation

* In a second => write operation 1 M / (24*3600) = 12 files/s
* In a second => read operation approximately 60 files/s.

* If we assume that average file is 100 KB. So in one day => 100 KB * 1 M = 10 GB, in 5 years = 10 GB * 30 * 12 * 5 = 18000 GB = 18 TB in 
5 years. Additionally, for unique URL we can use some key service that generates unique keys. Key generation services create the unique 
strings and store the data. For Key Generation Services capacity = 6 character * 8 bytes * (base 64 encoding 64 ^ 6 = approximately 60 
billion) = this is negligible

We can use the approximation of usage total capacity. We use maximum %70 capacity at the same time. So we need to 25 TB storage.

### Bandwith estimation

for write operation => 12 * 100 = 1.2 MB/s
for read operation => 60 * 100 = 6 MB/s

### Caching estimation

For caching we can use %20 rules. This means that in a day 10 GB / 5 = 2 GB caching can be used.

## System API's

* We can basically prefer REST API for this website. Basically three API first is uploadFile(key, URL, text, private = none) return 
uniqueURL, second one is readFile(key, URL) redirect original URL, third one is deleteFile(key, URL) return HTTP response(SUCCESS or FAIL)

### DATABASE

* We can prefer NoSQL databases like Dynamo or Cassandra. Because there is huge data and no relationship between URLs, this is the good 
option. We can quickly obtain the data by using NoSQL. Basically, three table should be. First one is File, User, File and User.

## Let's first think about basic system design

* Storage can be divided into two parts.
a-) Metadata storage
b-) Block storage
c-) Client
d-) Application server

* Think about system design more deeply

Metadata storage
Block storage
Clients
Load balancer (client and application servers, application servers and block servers, block caches, metadata servers, metadata caches)
Key generation service data storage
Block cache
Metadata cache
All services have two or more replica servers.

### REPLICATION AND DATA PARTITIONING

We can use sharding (horizontal partitioning for this system). In addition, data replication can be used. (Each server has two or more 
copy server). The hash-based partition should be used to provide balancing.

### CACHING

Notice that we use %20 rules for caching. In addition, block storage cache and metadata storage cache can be used. an LRU mechanism 
can prefer for caching

### LOAD BALANCER

We can use Round Robin principle but this principle has an important problem. This cannot handle if server is busy or fail, it 
continue to redirect busy or fail server. We can create more intelligent load balancer principle for this problem.
    
