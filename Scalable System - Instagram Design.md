### WHAT IS INSTAGRAM?

* Do you really know the Instagram? Instagram is one of the best social media platforms in today's world. 
Most of the people use the Instagram every day effectively and continuously.  This causes that Instagram is under heavy-traffic. 
As you know, heavy-traffic means that there are too many incoming requests to the system and system should respond all these requests 
under reliability, availability and minimum latency.
As you know Instagram is a social media platform that ensures users to upload, share, view, comment pictures via this platform. 
Additionally, you can follow other users. If you are using Instagram, you can realize that user's timeline is one the best and 
important topic for Instagram because every user meet this system in its timeline and timeline creation time is so important under 
this conditions.

### REGUIREMENTS AND GOALS OF SYSTEM?

* As you know, if you need to design your design carefully, you should focus three requirement topics which are functional, nonfunctional,
extended requirements.

### Functional Requirements

* Users can register the system.
* Users can log in and log out the system.
* Users can share, download, view pictures of their pictures or other user's pictures.
* Users can follow other users.
* Users can share (upload) pictures when they register and log into the system.
* Users can view (download) pictures when other users allow that their pictures can be viewed publicly.
* Users can view hottest pictures in their timeline.
* Users can search pictures based on titles.

### Nonfunctional Requirements

* The system should be highly available.
* The system should be highly reliable. As we said, users can upload pictures and all data of users should not be lost.
* The system can work based on CAP Theorem. If we talk about a system like Instagram. Transaction operations are not as 
important as financial operations. This means that consistency can take time. This is sufficient if the system will be 
consistent within a certain period of time. Availability is more important than consistency so a user may not see photo for a 
while, this is fine.

* As we said, timeline creation process is one of the hardest and important points of the Instagram design. This is good 
if the optimum time for creation timeline is 250 ms.

### Extended Requirements

* The system should monitor.
* The creation of timeline needs to effective algorithms for decreasing time.
* We can realize that this system is read-heavy so we should focus on the uploading and present pictures, so reliability 
and minimum latency are two main points of consideration.

**Note: Instagram is a huge system and I didn't deal with the comment process, recommendations and tags. Recommendation 
systems are another important system that it should be designed carefully.

### CAPACITY AND ESTIMATION

When we talk about the define capacity and estimation we should think the future of the system. To illustrate this, we can deal with 
the data that collect up to 5 or 10 years. It helps us to scale easily. Let's assume the total user count of Instagram is 500 Million 
and the daily user is 10 Million. If we assume that each user uploads 2 picture in each day, daily incoming picture count is 20 Million.
Notice that we can think the average size of the picture is 500 KB so total required space for 1 day is 
20 Million * 500 KB = 10 * 10^12 = 10 TB. Total space required for 5 years can easily be estimated. 
Note that, this capacity doesn't contain replicate data. Additionally, we should just only use until 80% of total existing capacity.

### API DESIGN

** We can use REST or SOAP to server our APIs. Basically, there are three important API of our system.

* **UploadPicture (api_dev_key, picture, title, picture_description, tags[], picture_details)
  UploadPicture base on the uploading picture. api_dev_key is the API developer key of a registered account.**

We can eliminate hacker attacks with api_dev_key. This API returns HTTP response. (202 accepted if success)

* **DownloadPicture (api_dev_key, search_query, user_location, maximum_video_count, page_token)
  Return JSON containing information about the list of pictures. Each picture resource will have a picture title, creation date, 
like count, dislike count, total view count.**

* **DeletePicture (api_dev_key, pictureID) -> Return HTTP response if success.**

There are a lot of another APIs to design Instagram, however, these three APIs are more important than the others.

### DATABASE SCHEMA

** As you know, we have talked about the pictures and users basically. We have to decide whether to use SQL or NoSQL before defining 
database tables. We can use RDBMS to keep data but as you know, scale process of a traditional database system is hard when 
we decide to keep data o a traditional database system. On the other hand, if we use NoSQL, we can scale system easily. 
There are three tables to store data;

```bash
User
  UserID : Int
  UserName : Nvarchar
  UserRealName : Nvarchar
  UserSurname : Nvarchar
  Mail : Nvarchar
  BirthDate : DateTime
  RegisterDate : DateTime
  LastLoginDate: DateTime

Picture
  PictureID : Int
  UserID : Int
  PicturePath : Nvarchar
  PictureLatitude : Int
  PictureLongitude : Int
  CreationDate : DateTime

UserFollow
  UserID1 : Int
  UserID2 : Int
```

* If we choose the NoSQL to keep data, we need to add a new table system. (PictureUser). We can store photos in S3 or HDFS. We can store all information about pictures with a key-value pair like Redis. Key is pictureID, a value is other information about the 
picture. (For NoSQL)
* We can store all information about users with a key-value pair like Redis. Key is userID, a value is other information about the user.
(For NoSQL)
* We can use Cassandra, column-based data storage, to save follow-up of users.

* **Note: A lot of NoSQL database supports replication.

* **Note: We need to have an index on PictureID and CreationDate because we need to get hottest pictures.

### COMPONENT DESIGN

* We can realize that uploading and downloading operation are not same. Uploading operation is slower than downloading operation because 
uploading operation just based on disk. On the other hand, read operation could be faster if we are using a cache.

If a user tries to upload a picture, he/she can consume all the connections. This causes to when uploading operation continues, 
the system may not respond to read operation. If we divide uploading and downloading operations into two separate services, 
then we can handle this bottleneck. Notice that, web servers have connection limits at any time and we need to focus on this point. 
Notice that, separating of uploading and downloading operations ensure that system can be more scalable and optimize.

### HIGH LEVEL SYSTEM DESIGN

* If we are designing a system, the basic concepts we need are;

* Client
* Services
* Web server
* Application server
* Picture Storage
* Database
* Caching
* Replication
* Redundancy
* Load balancing
* Sharding

There are two separate services in this system, which are upload image and download image. Picture storage is used to keep pictures. 
A database is used to save all information about users and pictures. (metadata). When a request comes to the system, the first to 
meet request is the web servers. Web servers redirect an incoming request to application servers.

### REPLICATION AND REDUNDANCY

Replication is a very important concept to provide availability and reliability. As we said, Instagram should ensure that any files 
cannot be lost. Replication is a very important concept to handle a failure of services or servers. Replication and redundancy basically
mean the copy of services or servers. We can replicate database servers, web servers, application servers, image storages and etc.. 
Actually we can replicate all parts of the system. Notice that replication also helps system to decrease response time. You imagine, 
if we divide incoming requests into more resources rather than one resource, the system can easily meet all incoming requests. 
In addition, the optimum number of a replica to each resource is 3 or more. Thanks to replications, if any server dies, the system 
continues to respond via secondary resource.

### DATA SHARDING

As you know, sharding is a very important concept which helps system to keep data into different resources according to sharding 
process. There can be two sharding procedure to use. First is partitioning based on UserID and second is partitioning based on PhotoID.

Partitioning based on UserID: We can divide incoming requests based on UserID. We will find the shard number by UserID % number of 
shards. Conditioning with shard based on UserID causes to problems. First is that system may be non-uniform distributed and second is 
if a user is more active than the other user, the data of this user may not be fitted into one resource. Another possible problem 
is handling the PictureID creation. PictureID should be unique in the system, so each shard needs to have its creation policy.
Partitioning based on PictureID: We can divide incoming requests based on PictureID. We will find the shard number by PictureID% 
number of shards. We can handle the non-uniform distribution problem and popular user problem. We can easily create PhotoID with 
Key Generation Service. Key Generation Service creates unique identifiers at first then serve this unique identifier to incoming 
pictures. This helps us to handle PhotoID problem.

### CACHING

Cache memory is a crucial part of reading data faster. Cache memory usage can base on 80-20 rule. This means that cache capacity is 
the 20% of the daily data size. We can use LRU cache policy (Least Recently Used).

* CDN: CDN, Content Delivery Network is for distributed file cache servers. We can usage CDN for keeping pictures.
* Memcache / Redis: Keep metadata in the cache with Memcache or Redis.

### LOAD BALANCER

* Load balancer allows incoming requests to be redirected to resources according to certain criteria. We can use load balancer at every 
layer of the system.

* Between requests and web servers.
* Between web servers and application servers.
* Between application servers and databases
* Between application servers and image storages.
* Between application servers and cache databases.
* We can use Round Robin method for the load balancer. Round Robin method prevents requests from going to dead servers but 
Round Robin method doesn't deal with the situation that any server is under heavy-traffic. We can modify Round Robin method to be 
a more intelligent method to handle this problem.

### DESIGN CONSIDERATION

Notice that we need to get the popular, latest and relevant photos of other people that we follow. We can use a pre-generate timeline 
to decrease latency because you image that system will fetch all friends of us firstly then fetch all pictures of our friends. 
After that, the system will combine all pictures based on creation time, like count or other properties. This action can take time 
and causes to late response. Pre-generate timeline keeps users' timelines into a table previously. Thanks to the pre-generate timeline, 
the system can serve them without the hassle of processing when they need to. What needs to be discussed here is what to do when 
new data comes in. There are three important methods we can mention,

* Pull: The client asks if there are any changes at regular intervals. This creates some problems. To illustrate this, 
we may not get new data when we use the system. Another problem is most of the type, the client can encounter with the empty response.
* Push: In push method, a server can push new data to clients as soon as it is available. Long Polling is one the best 
methods to use this method efficiently. Long polling is a method that there is an open connection between the client and the server, 
and if any change occurs about data, server return response to the client as soon as possible. This method may cause a problem for 
users that follow a lot of users.
* Hybrid: Hybrid method is a combination of pull and push methods. Push method is for users that follow few users and pull 
method is for users that follow a lot of users.

### BASIC CODING OF THE SYSTEM

```bash
public class Server{
  ArrayList<Machine> machines = new ArrayList<Machine>();
}
public class Storage{
  ArrayList<StorageMachine> machines = new ArrayList<StorageMachine>();
}
public class Machine{
  public ArrayList<User> users = new ArrayList<User>();
  public int machineID;
}
public class StorageMachine{
  public ArrayList<Picture> pictures = new ArrayList<Picture>();
  public int machineID;
}
public class User{
  private ArrayList<Integer> friends;
  private ArrayList<Integer> pictures;
  private int userID;
  private int machineID;
  private String information;
  private Server server = new Server();
  private Storage storage = new Storage();

  public User(int userID, int machineID){
      this.userID = userID;
      this.machineID = machineID;
pictures = new ArrayList<Integer>();
friends = new ArrayList<Integer>();
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

  public void addFriend(int id){
      friends.add(i);
  }

  public void addPicture(int id){
      pictures.add(i);
  }

  public int[] getFriends(){
      int[] temp = new int[friends.size()];
      for(int i=0; i<temp.length; i++){
          temp[i] = friends.get(i);
      }
      return temp;
  }

  public int[] getPictures(){
      int[] temp = new int[pictures.size()];
      for(int i=0; i<temp.length; i++){
          temp[i] = pictures.get(i);
      }
      return temp;
  }

  public User lookUpFriend(int machineID, int ID){
      for(Machine m : server.machine){
          if(m.machineID  = machineID){
              for(User user : m.users){
                  if(user.userID = ID){
                      return user;
                  }
              }
          }
      }
      return null;
  }   

  public Picture lookUpPicture(int machineID, int ID){
      for(StorageMachine m : storage.machine){
          if(m.machineID  = machineID){
              for(Picture picture : m.pictures){
                  if(picture.pictureID = ID){
                      return picture;
                  }
              }
          }
      }
      return null;
  }
}
public class Picture{
  private int machineID;
  private int pictureID;
  private String photoPath;

  public Picture(int machineID, int pictureID, String photoPath){
      this.machineID = machineID;
      this.pictureID = pictureID;
      this.photoPath = photoPath;
  }

  public int getMachineID(){
      return machineID;
  }

  public void setMachineID(int machineID){
      this.machineID = machineID;
  }

  public int getPictureID(){
      return pictureID;
  }

  public int getPhotoPath(){
      return photoPath;
  }

  public void setPhotoPath(String photoPath){
      this.photoPath = photoPath;
  }
}
```
