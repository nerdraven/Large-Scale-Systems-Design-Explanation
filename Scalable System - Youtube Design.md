### WHAT IS YOUTUBE?

* Youtube is one of most popular video sharing system that presents for the people. Users can easily share, upload, view, rate, comment 
videos via the system. Users can register the system and use the system by their account. Youtube also is one the best systems that 
recommend videos to users according to their interest. This means that people can continue with the these suggested videos. 
Moreover, with having the special account on Youtube, users can follow other users or channels. Additionally, as we mentioned above, 
users can comment videos regardless of whether they are logged in or not.

### REQUIREMENTS AND GOALS OF THE SYSTEM

* As you know, Youtube is one the biggest system today’s world, however, we can design the main features of the Youtube. Although there 
are a lot of different hard process to design Youtube, regardless of the recommendation system, channels, my favourite videos feature, 
most popular videos, detailed search mechanism, we can design the Youtube according to other features listed below,

Note that, there are three types of requirements which are functional requirements, nonfunctional requirements and extended requirements. 
We should define all required steps before starting the designing operation. Also, we should be careful to estimate capacity at the 
beginning of the design because when we define all required and capacity steps literally, we handle problems we can encounter at the 
design steps.

### Functional Requirements

* Users can register the system.
* Users can login or log out the system.
* Users can share, upload, view, comment videos in the system.
* Users can find videos by searching the videos.
* Users can like or dislike the videos, under this condition, the system should be kept a number of likes, dislikes, comments,
views to present these number to users.

### Non Functional Requirements

* System should be highly reliable. As we mentioned above, users can share and upload videos to the system and system should 
guarantee to keep these videos against loss. Replication and sharding help system to keep data against loss. We will explain replication
in detail under step8.

* System should be highly available. If we talk about Youtube, Netflix or other large scalable systems, this means that these 
systems are going to be exposed to large traffic. There can be a huge number of request at the same time at the system and system should 
tend to respond to all requests in a real-time experience. Replication, sharding and load balancer helps system to be highly available. 
We will talk about all three features under step7, step8 and step9.

* System should respond with minimum latency. You imagine that you would be a user and you want to do anything on the Youtube, 
do you really want to wait to much time to get the response? I think you don’t want to wait too much time as well as you want to 
experience the system with real-time. To illustrate this, when you search a video in the system, this system should suggest related 
videos as soon as possible.

### Extended Requirements

* Extended requirements mainly for improving system performance. To illustrate this, the system can be monitored to define usability 
  and durability.

### CAPACITY ESTIMATION

To estimate the capacity of the scalable system, we should focus on the total user, active user, number of coming request and data. 
All these features can be considered for next 10 years because when we do that, we can easily handle the scaling problems that we can 
encounter after years. We can estimate the storage capacity, total usage, bandwidth estimate, cache capacity etc.. before starting the 
design process. Notice that, at systems like Youtube, the number of the read and write operations can be different. For example, in our 
system, we can estimate that the ratio of the write and read operation is 1:100. This is briefly called an upload and download ratio.

### User estimation

We can assume that total user count of the Youtube is 1 billion and Youtube has 500 million active users in each day.

### Storage estimation

Let’s talk about the average size of videos. We are just focusing on the uploading operations because only uploading operations take up 
space at the storage. We assume one minute of video needs 50 MB to storage. This also should include the multiple formats of the 
uploaded video. Let’s try to find the total minute of uploaded video in each day. As you remember, we have mentioned the ratio of write 
and read operation which is 1:100. If we assume that each active user downloads 5 videos then the number of uploaded video is 
500 Million * 5 / 100 = 25 Million. We can assume that each video is approximately 2 minute. A total video minute is 50 Million in each 
day.  As a result, daily needed storage approximately equals to 
50 Million * 50 MB = 125 * 10^6 * 50 * 10^6 = 125 * 50 * 10^12 = 6250 * 10^12 = 6250 TB. This amount of daily storage is huge. 
Note that, this calculation doesn’t contain the replication, duplication and etc… and because of that the storage estimation can 
change and may be overestimated.

### Cache Estimation

Cache is the fast way to catch the data. I have explained the cache system in this link Caching. You can get the detail information of 
the cache system.
In general, a cache is estimated according to daily storage. To estimate the cache capacity, we use the popular ratio based on the 
80-20 rule. This means 20% of popular data are cached. Briefly, we can estimate that Youtube cache capacity approximately equals to 
6250 TB / 5 = 1250 TB.

### Bandwith Estimation

An internet connection with a larger bandwidth can move a set amount of data much faster than an internet connection with a lower 
bandwidth. Bandwith is calculated according to both of download and upload operations for a second.

### DATABASE DESIGN

* There are two choices to define the database schema. These are SQL and NoSQL. We can use traditional database management system like MsSQL or MySQL to keep data. As you know, we should keep information about videos and users into RDBMS.  Other information about videos, called metadata, should be kept too. Now we have main three tables to keep data. (Notice that we just only think the basic properties of Youtube. We can forget the recommendation system).

```bash
  User

    * UserID (primary key)
    * Name (nvarchar)
    * Age (Integer)
    * Email (nvarchar)
    * Adress (nvarchar)
    * Register Date (DateTime)
    * Last Login (DateTime)

  Video

    * VideoID (primary key – generated by KGS – Key generation service)
    * VideoTitle (nvarchar)
    * Size (float)
    * UserID (foreign key with User Table)
    * Description (nvarchar)
    * CategoryID (int) : Note that we can create Category Table to define categories
    * Number of Likes (int)
    * Number of Dislikes (int)
    * Number of Displayed (int) – We can use big int to keep displayed number
    * Uploaded Date (DateT?me)

  VideoComment

    * CommentID (primary key)
    * UserID (foreign key with User Table)
    * VideoID (foreing key with Video Table)
    * Comment (nvarchar)
    * CommentDate (DateTime)
```

### API DESIGN

* A lot of systems support mobile platform so APIs are the best choices to be able to provide the distinction between 
developers and support mobile support as well. We can use REST or SOAP. A lot of huge companies prefer to REST or SOAP according to 
their systems. There are three main API’s we will mention below:

* UploadVideo(apiKey, title, description, categoryID, language)

Upload video is the first API that we should mention. There are basically five main properties of this API. You can add more 
properties to UploadVideo API. Note that, apiKey is the developer key of registered account of service. Thanks to apiKey we can 
eliminate hacker attacks. UploadVideo returns the HTTP response that demonstrates video is uploaded successfully or not.

* DeleteVideo (apiKey, videoID)

DeleteVideo returns HTTP response like that demonstrates video is deleted successfully or not.

* SearchVideo (apiKey, query, videoCountToReturn, pageNumber)

SearchVideo takes 4 main properties and you can add more properties to SearchVideo API. SearchVideo returns the related videos 
according to query. We should not forget that SearchVideo is based on a title of the video. SearchVideo API returns the JSON that 
includes the information of related videos. Notice that this API can return the likes count, dislikes count, comments count 
and etc…

If we talk about the real Youtube service, SearchVideo API considers the recommendation.

### HIGH-LEVEL DESIGN PROCESS

There are basic features found in web-based systems. The main ones are client, web server, application server, database and cache 
systems. Depending on the intensity of system traffic, the number of servers or services increases and the load balancer distributes 
incoming requests between these servers or services. Additionally, queues can be used depending on the density of incoming requesters. 
Queue operation helps users to keep from waiting more time to get respond. In our Youtube service;

```bash
  – Client
  – Web Server
  – Application Server
  – Database
  – Video Storage
  – Encode
  – Queue
```
We can distribute services to three parts to decrease response time because video uploading takes more time from video downloading. 
Video can be downloaded from the cache and getting data from the cache is a fast way. The client basically users who use the system. 
Web Server is the first entity that meets request. Incoming request can take place in upload service, search service or download 
service. If we give an opportunity to users that download video asynchronously,  system traffic will be relaxed.  An encoder is to 
encode uploaded video into multiple formats. There are three types of databases which are Video content database, user database and 
video metadata storage. Queue process takes place between an application server and encode.

Our Youtube service would be read-heavy and we should be careful when building a system. Our main goal should be returning videos 
quickly. We can keep copies of videos on different servers to handle the traffic problems. Additionally, this ensures the safety of 
this system. We can distribute our traffic to different servers with using a load balancer. The system can keep two more replicas of 
metadata database, user database and video content database.  We can use CDN to cache the popular data.

* Flow diagram of the system;

* A client sends a request.
* Request meets from the web server.
* Web server controls the cache. There can be two more cache databases on the system.
* If the request takes place into a cache, a response is redirected to the client.
* Otherwise, web server redirects the request an to the application server.
* There can be load balancer between web servers and application servers.

** If this request is search or view service, it tries to find the request by looking at the metadata database and the video content 
database. A load balancer can be placed each layer of the system such as between application server and video content database, 
between the application server and metadata database etc…
When a server responds the request to the client, related data is cached according to the cache process.

### OPTIMIZATION OF THE SYSTEM

As we know, Youtube is huge video sharing system. Users can upload videos and the number of uploading videos grows exponentially day 
by day. According to uploading videos, there may one more same video in the system. To eliminate the duplication of videos we can 
implement an intelligent algorithm. For example, when a video comes to a system, the algorithm automatically checks whether this video 
is already kept in the system or not. If the system has already this video, then we don’t need to keep duplicate data. It saves us from 
unnecessary use of space. Additionally, if incoming video includes a video kept in the system, then we can divide videos into small 
chunks and we just give the only reference to same video chunks to handle the duplication problem.

### SHARDING PROCESS

As we mentioned above, sharding ensures that the system is more reliable and available. There can be two types of sharding process 
which are userID based and videoID based. Notice that userID based sharding process may cause problems. To illustrate this, when we 
shard data according to userID, the system tries to fit all user data in one machine and if the user uploads huge video to Youtube, 
we can face a fit problem. Another problem for sharding based on userID is a uniform distribution. Not every person uploads the same 
data size. This causes that system can be distributed non-uniformly. When we shard data according to videoID, we can eliminate these 
two problems.

### REPLICATION PROCESS

As we mentioned above, the replication process is a valuable process to provide high availability, high reliability and real-time 
experience. As you know, it is recommended to have 3 or more replicas for each server and service. If our system has 2 more replicas 
for an application server, web server, CDN, encoding service, user database, metadata database, cache DB, system automatically becomes 
more reliable and available. We can keep same data onto three different resources and thanks to this process if one server dies, 
the system automatically continues to work replicas. One more advantage of replication is the system may continue to run at an update 
to the system.

### LOAD BALANCING

Load balancing refers to distributing requests across a group of services and servers. When we have talked about the replication and 
sharding, an incoming request must be directed and this is done by a load balancer. We can use Round-Robin method to redirect incoming 
requests but there may be a problem in this case. Round-Robin stops sending requests to dead servers but Round-Robin cannot stop 
sending requests to a server exposed to heavy traffic. We can prepare intelligent Round-Robin algorithm to take away this problem. 
Additionally, we can use consistent hashing to redirect incoming requests. Consistent hashing ensures that system becomes more 
uniformly distributed.

### CACHING MECHANISM

There can be two types of caching mechanism in our system.

* Memcache (for user and video database)
* CDN (for static media)

** Memcache or Redis is the good way to create cache mechanism. As you know, caching ensures that system responds popular data quickly. 
Additionally, we can set up a caching mechanism on same servers or different servers. We should be careful about caching mechanism 
because when we keep cache data on the same server, this means we have limited capacity. On the other hand, when we set cache mechanism 
on different servers we have more capacity. However, using a different server for caching mechanism cause latency problem. LRU 
principle is a good way to set up a cache mechanism.

### DESIGN CONSIDERATION

Video uploading is a big process. When it fails, the system should ensure that it should continue to upload video from the failing point.
Video encoding process should include the queue operations. When an uploaded video comes, this new task is added to a queue and all tasks 
in the queue are taken one by one from a queue.

### BASIC CODING OF SYSTEM
```bash
public class Server{
    ArrayList<Machine> machines = new ArrayList<Machine>();
    }
}
 
public class Storage{
    ArrayList<VideoMachine> machines = new ArrayList<VideoMachine>();
}
 
public class Machine{
    public ArrayList<User> users = new ArrayList<User>();
    public int machineID;
}
 
public class VideoMachine{
    public ArrayList<Video> videos = new ArrayList<Video>();
    public int machineID;
}
 
public class User{
    private ArrayList<Integer> videos;
    private int userID;
    private int machineID;
    private String information;
    private Server server = new Server();
    private Storage storage = new Storage();
 
    public User(int userID, int machineID){
        this.userID = userID;
        this.machineID = machineID;
        videos = new ArrayList<Integer>();
    }
 
    public String getInformation() {
        return information;
    }
 
    public void setInformation(String information){
        this.information = information;
    }
 
    public getID(){
        return userID;
    }
 
    public int getMachineID(){
        return machineID;
    }
 
    public void addVİdeo(int id){
        videos.add(i);
    }
 
    public int[] getVideos(){
        int[] temp = new int[videos.size()];
        for(int i=0; i<temp.length; i++){
            temp[i] = videos.get(i);
        }
        return temp;
    }
 
    public User lookUpVideo(int machineID, int ID){
        for(Machine m : storage.machine){
            if(m.machineID  = machineID){
                for(Video video : m.video){
                    if(video.videoID = ID){
                        return video;
                    }
                }
            }
        }
        return null;
    }
}
 
public class Video{
    private int machineID;
    private int videoID;
    private String videoPath;
 
    public Video(int machineID, int videoID, String photoPath){
        this.machineID = machineID;
        this.videoID = videoID;
        this.videoPath = videoPath;
    }
 
    public int getMachineID(){
        return machineID;
    }
 
    public void setMachineID(int machineID){
        this.machineID = machineID;
    }
 
    public int getVideoID(){
        return videoID;
    }
 
    public int getVideoPath(){
        return videoPath;
    }
 
    public void setVideoPath(String videoPath){
        this.videoPath = videoPath;
    }
}
```
