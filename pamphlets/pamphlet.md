## 0 - Computer Architecture
Disk = storage = hard drive(HDD hard disk drive, nowadays SSD is more common)

Memory = RAM(random access memory)

Reading and writing(done by CPU) from RAM is measured in terms of microseconds(1/10^6) (latency) and it's a lot faster than reading and
writing from disk which is measured in terms of milliseconds(1/10^3). So RAM is a lot faster than disk.

What if we want to communicate between RAM and disk? These components can't directly talk to each other, that's why we have to introduce
another component which is CPU(brain of computer).

The main things that CPU is capable of:
- reading and writing info to RAM or disk
- executing our code

While reading and writing from RAM are quick(microseconds) what if we wanna go even faster?

CPUs have another component called the cache which is a lot smaller than RAM(RAM itself is a lot smaller than disk). It's typically measured
as megabytes and is used to speed up read and write operations from RAM. It's measured in nanoseconds, so it's a lot faster.
Cache is mainly used as a substitution for the RAM.

What CPU will do is it'll take a subset of RAM, one portion or multiple portions that can fit within our cache and we want to choose
these portions intelligently. It will choose portions of the RAM that we're reading from or writing to, very frequently. Because we have
a limited amount of space in cache.

It's not necessary to have a cache but it will make things a lot faster.

If the CPU wants to read some data from RAM, it can first check if that data is in cache, if not, it will perform the more expensive operation
which is reading from RAM. But maybe it will throw it in cache so next time we have to read it, we can directly go to cache for it.

RAM and cache don't have persistent data.

The reason we have a cache, is to lower the latency which can increase the overall throughput of system. So our CPU can handle more
operations per time.

## 1 - Application Architecture
An application architecture for a production app.

The problem we introduce when we have multiple servers handling reqs is that when a user makes a req, how do we know which server
that req should go to? We use load balancer. It'll forward req to the server that has the minimum amount of traffic.

We don't store **log statements** on our server. Because we don't need to because our server isn't really gonna be interacting with them,
instead, we as developers are going to interact with the logs, so we will have external service that is gonna store our logs for us.

What if one of our servers isn't running well, maybe it has a faulty CPU. We want to know about the resources that our server is using.
Is our server responding to all reqs? Or some reqs are failing? To get that insight, we would have a **metrics service**.

Some of the metrics might directly come from logs. So log-based metrics.

We could use logs to create metrics.

Metrics are usually displayed in terms of time series charts. Logs-based metrics can also be used because logs are naturally time series data.
Everytime we log sth, it has some timestamp associated with it.

As devs, we can look at the metrics and get insight into how the app is running, but if sth goes wrong, you wouldn't want to have to **manually**
go and look at the metrics to realize what's going on. Or even worse, a user would tell us like emailing us that sth went wrong with our app.
We don't want this to happen. We(as devs) want to know immediately when sth goes wrong, so we want a push notification from the metrics.
To accomplish this, we have our metrics feed data into an alerting service which will for example tell us when a metric has reached
some threshold.

logging -> metrics -> alerts

These components are running on different computers. So there's some network component between them.

## 2 - Design Requirements
### design requirements
1. move data: Can be moving data from a component of machine to another component or from one data center to another data center.
   Moving data isn't so simple when machines are located at different data centers. We have to move data across networks
2. store data
3. transform data: For example, given a bunch of server response logs data, maybe we want to aggregate that data and transform it to find out
   what percentage of them was successful? and what percentage was failed?


These 3 operations encapsulate all of the functionalities of most apps.

How do we know if our app is designed well? Because there's not always a correct answer.

We have to think in terms of tradeoffs. How do we measure tradeoffs?

Availability: uptime / (uptime + downtime)

We measure availability in terms of 9s. Five 9s is a great availability. But for example 99% is not very good in a year it means 3.65 days it's
down.

Availability is used to define SLOs(service level objective). As devs we define some goals like availability, these are called SLOs.
SLA is different than SLO. SLO is sub portion of SLA.

As AWS, we would define SLA for our DB. It would be the SLO which is we want our DB to have 5 nines of availability. But as SLA, we say
if we don't reach the level of availability(determined by SLO), we will give you a partial refund. It's a AWS policy actually.

SLA is what customers can expect of our service and what's they pay for. It's a customer agreement. Service level agreement. It's not just a goal,
it's an agreement with a customer and says: this is what you can expect or these will be the consequences.

---

### Reliability
Systems can have reliability, full tolerance and redundancy. These contribute to each other. They have some differences.

If a user makes a req to our server and it responds, that means our server was available but doesn't necessarily mean our server is reliable.

Reliability is the probability that our system won't fail. We know if we just have a single server that's responding to users, we have a
higher probability of failing than if we have 2 servers. So by adding a server, we can increase the reliability of our server, but we're also
increasing the availability of system because for example if we have a single server, a lot of reqs could be sent to it or we can have
people maliciously trying to take our server down(DDOS attack - distributed denial of service attack).

**Q:** Vertical scaling increases availability because a higher percentage of users would get a response. What about reliability?

We still only have a single server, so reliability doesn't change. This is why horizontal scaling can have benefits compared to vertical scaling.

### fault tolerance
If one portion of our system has a fault, the system continues to operate successfully. One server goes down but the second server still works.
So fault tolerance is related to reliability.

### redundancy
We add a server that's running the exact same code that we don't need it in order to respond to users. Because the other server is capable
of doing everything. But it helps to have both because in the event of a fault, we have multiple copies, so our system continues to function because
of redundancy.

By having redundancies in our system, we're able to have fault tolerance and with fault tolerance, we can provide higher reliability.

Having the redundant server in same datacenter, we could have a disaster that goes down the both servers. So it would be better to have
the second server in a different datacenter or maybe other part of the world.

---

### Throughput
Means the amount of operations or data or sth we can handle over some period of time. In the context of communicating with a server, we would measure
the throughput of this, in terms of **reqs per second**(request/second). This means how many concurrent users, could our system
handle per second.

If we can horizontally or vertically scale, we can handle more reqs.

But when we only scale vertically, that server is a **single point of failure**. If it goes down, our entire system
goes down. So we need more reliability, fault tolerance and redundancy.

If we scale it horizontally, the downside is adding more complexity. Because we have to balance the reqs between servers. So we need a
load balancer which increases the complexity.

Horizontal is less limited.

When we talk about how many reqs a DB can handle(throughput of DB), we usually measure it in terms of queries per second(queries/second) or
QPS and it's similar to servers. We can even call it reqs/second ! Query is just more used when talking about DBs.


DBs have the same problems as servers. Where a single DB can be a single point of failure. Horizontal scaling vs vertical scaling.
In horizontal scaling of DB we need to keep data in multiple DBs in sync.

A third measurement of throughput is **amount of data per second** usually measured in bytes/second . This is a variation of reqs/second or
queries/second.

In servers, it makes more sense to measure it in terms of reqs/second because each req is mapped to a single user.

But in a data pipeline, where we given data and we wanna process data into some other format and it's not necessarily related to a single
user, it's better to measure throughput in terms of bytes/second. Because maybe we don't even have a req.

### Latency
It's just some period of time. In other words, amount of time it takes for an operation to complete. In req-res model,
the latency would be the amount of time it takes for that operation to complete(until we get a res back).

Latency is how long each **individual** req takes. But throughput is related to multiple reqs.

To lower the latency of a distributed system, we could introduce a second server and we could have these servers on different sides of world.
This increases the availability and reliability and throughput, but it also reduces the latency for users located on different sides of world.

Another technique for reducing latency is having CDN.

## 3 - Networking Basics
An IP address is a 32 bit integer and you have 12 digits in total: 012.345.678.910

Since it's 32 bit, we can't have all of the digits go up to 999. Because that wouldn't fit in a 32 bit integer. So the max value for each of these
is gonna be 256. So the biggest IP address we could contain: 256.256.256.256 (at least when you look at it as a single integer)

With 32 bits we can store up about to 4 billion unique public IP addresses. This is limiting. This limitation is related to IPV4.

IPV6 allows up to 128 bits.

Every machine is gonna be uniquely identifiable with an IP address.

What are the rules of communication between computers?

An analogy would be sending mail in real life. We need to specify From and To(metadata).

So the data itself(packets) has to include which IP address it's going to and which IP address this data came from?

A portion of a packet is gonna be reserved for that From and To and ... info which we call them metadata. That metadata is what describes
that entire packet. We call that metadata a IP(internet protocol) header. So a portion of packet is gonna be **IP header**.

The rules of sending data over internet is called the internet protocol(IP).

IP allows us to send data(packets) between machines but it doesn't do everything we need it to do.

What if instead of just sending one letter in the mail, we wanted to send an entire book in the mail? Let's assume we want to do so in an
envelope instead of a box.

One way to do is to rip every single page of the book and put it in an envelope and let's say it takes us 10 envelopes to contain the entire book
and we send those 10 envelops from same person to other person.

We can do that with internet protocol. But how does the other person reassembles the other pages(assume there is no page number)?

In IP there is no such thing for that. This is what another protocol called TCP solves that and other problems.

In TCP, in a packet, the portion that is not the IP header, is actually further broken up into 2 portions:
- TCP header: Some metadata about the TCP packet
- data

Recap till here: We're sending IP packets which have an IP header which allows us to communicate between computers. But when we're sending
large amounts of data we can't put all of that data within a single packet. So we have to split it up to multiple packets.

What kind of info would the TCP header includes that the IP header wouldn't?

One thing is sequence number of that packet. When the server receives all the packets, it's able to reassemble them in the correct order.

### Public vs private IP address
For a server to be publicly accessible, it needs to have a public IP address. But our client machine doesn't need to have a public address,
nobody's necessarily gonna be making reqs to it and actually the internet traffic that we use as clients, will go through some router.
So if we have multiple devices connected to a router, each of the devices doesn't need to have a public IP address, only our router does and
that's what happens. Router will in our local network(LAN), assign each of the machines connected to it, a private IP address which won't be
accessible on the internet.

### Static vs dynamic IP addresses
Static IP addresses are important on server. Because if clients are making a req to a machine, that IP address of server should be static.
But servers can also have dynamic IP addresses and things will continue working and that's because of **dynamic DNS**.

But for clients we don't need static IP address(in our local network, we don't have static IP address).

### Ports
It's a 16 bit value so 2^16 which is about 65000 ports for a machine.

Many application layer protocols have a default port. So when we use https to communicate with a server, we don't need to specify the port(443 in this
case). Because the default port for https is 443.

So from perspective of client, we send req to a specific port of server, but when server send res back using a port, that doesn't necessarily need
to be port 443, actually from client's perspective, it doesn't matter which port we receive that info on. So the protocol will randomly assign
a port and our browser will reserve that port to receive the response.

localhost is special name for IP address of 127.0.0.1 . This is sort of a reserved IP address which points at your local machine. So this is a
way of accessing a server that's running locally on your own machine.

## 4-4 - TCP and UDP
Internet protocol suite is commonly referred to as TCP/IP(TCP is running on top of internet protocol). But internet protocol suite also includes
other things that are not just TCP and IP, which includes UDP.

TCP is more common than UDP.

### TCP
When breaking data into pieces and sending it and then re-assembling it on other side, the order matters. This process is a property of 
TCP and doesn't come with internet protocol. So TCP handles the ordering of packets.

Another benefit of TCP is it's reliable. The reason this is important is that networks in general are not reliable.

Internet protocol doesn't have a solution for when a packet doesn't arrive at destination. But TCP does. With TCP, those packets(and not
the successful ones) will be re-sent(re-transmission of lost packets).

You don't get reliability or re-transmission with UDP.

Part of the reason of why this is(re-transmission) possible, is because TCP establishes a connection between the two machines that are communicating.
It's a 2-way connection. So when we establish a TCP connection, we can send data in both directions. To establish this connection, we need
3-way handshake which is part of why TCP can be reliable, why we can send data in both directions and why we can reassemble data at the destination,
but this is also expensive. So TCP has a lot of overhead because of guarantees it give. TCP is reliable but slower compared to UDP.

### UDP(user datagram protocol)
**Datagram** is the word for packet that's used with UDP.

The benefit of UDP is we don't need to establish a connection between the client and server, so there's no handshake.
There's no guarantee every packet will arrive. Also the data might arrive out of order.

UDP is a lot faster than TCP.

UDP is great for live streaming, online games. UDP is also used in DNS.

## 5-5 - DNS (Domain name system)
DNS is a decentralized system

```shell
# to get IP address of google.com
nslookup google.com
```
Now paste the IP address in browser and you will go to google.com .

So we have a decentralized system that takes a domain name and translate it into an IP address.

ICANN is owner of all domains in world. Why I can't steal google.com ? Who owns google.com? Well it has to go through ICANN. ICANN
doesn't sell domains since it's non-profit.

The sellers of domains are called domain registrars like google domains, namechip, godaddy and ... . These resellers host servers which have
the DNS records. So eventually a user request should land to one of these servers which will do the translating for us.

DNS record stores info about resolving a DNS request and there are many types of DNS records.

A record(address record): specifies the domain points to which IP address.

Since the IP address usually aren't gonna change for servers(usually they're static IP addresses), though static IPs are not required(in this case,
dynamic DNS takes care of this), the client can cache these IP addresses. So we don't need to go through the DNS query every single time. We
already have the IP address cached.

A server is a computer with a public IP address and typically that IP address has some domain name associated with it.

.com is top level domain(TLD).

The domain owner controls the subdomain entirely.

## 6-6 - HTTP
Client means the one that's making the req(initiates req). Client could be another server but still it initiates the req.

### RPC

### HTTP
HTTP is a request-response protocol. Client makes a req and server sends back response. HTTP is built on TCP(so under the hood there's a TCP
connection).

GET is expected to be idempotent. We're expecting that resource to not change when reading it. It's cachable. Also it means if we pass in
the exact same params and query params, we'll get the exact same results. It should not change as a result of GET req. This is important for
caching. But POST is not idempotent, so not cachable(after the first time issuing a POST req, it will continue to
do some action on server side). DELETE usually is idempotent, so it's more cachable.

### SSL/TLS
SSL came before TLS. These, encrypt everything we send over HTTP.

HTTP is not necessarily secure. It can't mitigate man in middle attacks.

## 7-7 - WebSockets
HTTP default port is 80.

### Limitations of HTTP
Some of the application layer protocols:
- websocket
- FTP(file transfer protocol): by default runs on port 21
- SMTP(secure mail transfer protocol): port 25
- SSH
- webRTC

All of these protocols are built on TCP(so reliably is important for these protocols) but webRTC is built on UDP.

In chat, we can send HTTP req to get back the new data between the last time we got back the response and now. Like sending HTTP req every 1 minute.
This is called polling.

How do we time these intervals?

60seconds is a lot. Maybe we can pull every 1 second. This would work but it's not optimized. We have to create a new HTTP connection(built on
top of TCP) everytime we want the new messages(every 1 second).

A better way is to use websocket that is built to solve this problem.

First we do websocket handshake which is we send an HTTP req like GET and that req is to establish a websocket connection and after that the server will
respond with 101 status code which means it's taking this connection and upgrading it to a websocket connection(101 means switching protocols).
After this handshake, there's a **persistent** connection established between client and server. We don't have to keep checking(polling) the
server for new content, everytime there's a new message, we immediately receive that from server, the server pushes them.
It's a bi-directional(full duplex) communication.

HTTP can't quite do this until HTTP2 which introduces streaming which kinda make websockets obsolete.

The tweet feed can be implemented with polling but in twitch chat, websocket is better.

The downside of websocket is there's a state established between the client and server. Maybe the connection would break at some point and 
then the server would continue to send data!

## 8-8 - API Paradigms
3 modern paradigms:
- REST
- GraphQL
- gRPC

### REST
**REpresentation State Transfer.** REST is not a protocol. Built on top of HTTP typically, but this doesn't mean REST is a protocol, it's
more a set of loose restrictions(standardizations).

REST apis are stateless. Because when we send reqs, we want to send everything that we need to know about that req and res.

An example of having state:

When going to next page, we need another 10 results. There are 2 ways we could implement that with our client-server architecture:
- server could have some info for every user, for example storing a session for every user on the server, so it will remember user saw
the first 10 results, so now it should send 10 next results. What if we had multiple servers? Maybe the other server doesn't
have the session info of a user. This is stateful. But we want REST apis to be stateless.
- for server to not have to know anything about ever user. The req should have all the info we need.

Note: In REST, the state is not stored on server but on client, there's definitely going to be some state in the browser. Like cookies,
session storage or ... .

When we make it stateless, so the servers don't need to store any state info, we allow ourselves to scale horizontally. We can add many servers
we want with a RESTful API. That's the big benefit of REST apis.

We don't add verbs to REST endpoints. Because the HTTP method like GET or ... is what we use as verb or what action are we taking on this resource.

The most common data format for REST APIs is JSON. It's readable but performance-wise, we can do better(gRPC).

### GraphQL
GraphQL is not a protocol, it's built on top of HTTP. But it only uses HTTP POST method, because we need to send data in body of req and there,
we include a query which tells which resource we want from server.

POST reqs are not idempotent in HTTP. So the downside is we can't cache graphql reqs as precisely as we can with REST APIs.

The problem it's trying to solve that we have in REST APIs:

- over-fetching: for example we just want the profile picture and username but when fetching the user, it will have a lot of info. This can be 
fixed in REST apis by adding custom logic on server to only send back the exact fields. With GraphQL we can specify exactly which fields of 
that resource, we want. So our app would be faster.
- under-fetching: Let's say for a page, we need to get the video and comments and the users that written those comments. So we need to make a
bunch of consecutive reqs. It's easier to group all of these reqs in a single req.

With graphql we would only have a single endpoint but there are 2 operations that we can do with graphql endpoints:
- query(aka doing reading, we don't modify any data)
- mutation(creating, updating, deleting, that would all fall under mutation)

Unlike REST apis, graphql does have schema for endpoints. We have a schema that we have to follow. We can only choose what's available. But in
REST apis which uses JSON, we can add anything to the body of req.

### gRPC
Not a protocol.

Built on HTTP/2 . Because it needs certain features of http/2. GRPC can not be used natively from a browser. If you want to make reqs from a 
browser using GRPC, you need a proxy. Like grpc-web. The reason we can't use it directly from a browser is grpc needs some fine grained control
over http/2 which browsers typically don't provide. So this is a downside. So grpc is used in server to server communication.

It's faster than REST APIs because instead of sending raw json(string), in grpc, it sends info in protocol buffers(schema objects) and
are serialized into binary. So we're sending a smaller amount of data.

GRPC also provides streaming, we can stream from client to server or from server to client or bidirectional streaming like websockets.
So grpc can do what websocket can do(http2 makes websockets obsolete).

The downside is a lot less tooling and less standardized.

Even though GRPC is built on top of HTTP and we know HTTP does have status codes, but in GRPC we don't have that, instead we have error messages.
So you have to have your own custom error handling based on error messages that you define server side.

In GRPC, every rpc or endpoint has an action or verb associate with it. So it's not just resource. Even though it's built on top of http(it has
http methods), we don't care about verbs and status codes.

## 9-9 - API Design
When you introduce non-backward compatible changes, you need to increase the version. So we don't break the apps of our clients using our APIs.

## 10-10 - Caching
With caching we lower latency and increase throughput(because it's faster to read and write from cache so we can read and write more data at
a faster rate).

`cache ratio = total number of hits / (total # of hits + total # of misses)`

Client caching and server caching

Getting data from DB is reading data from disk, so it's slow. We can add caching to server. We can take a subset of data on disk and put it
in memory(we can use redis). First we look at memory and if it was not there, get it from disk and also store the data in memory, so next time
somebody asks for the exact data, we look at memory first and it's there, that's a cache hit.

Even though our in-memory cache is much smaller than disk, since the vast majority of reqs want the data we stored in memory, this
speeds uo our server performance and increases the throughput. Because maybe reading from disk is 10000 reads / second but from memory
we can do a lot more. This can scale much higher.

There are different algorithms in caching that can accomplish different things:
- write-around caching: When there's a write req, it skips the cache entirely, we just write the data to primary storage(DB data that are stored in disk)
and then when we wanna read(we have a read req), we'll check the cache, data is not there for the first time, so we expect to be a cache miss the
first time we're trying to read data and then we find it on disk, throw it in cache and return it to user. Subsequent reads will read from cache.
In this approach, there's gonna be a significant number of cache misses, but at the same time, we're not gonna put anything in the cache, unless
it's actually being read.
- write-through caching: When we're creating some resource(write op), we immediately write it to cache **and** after that we write it to primary storage
which would be on disk. We're caching everything
- write-back caching: much faster way of doing things when possible, but it can be less reliable and cause some inconsistencies. It's a lazy
way of doing things. When we wanna create sth(create req), we would throw it in memory(cache) and we would skip the disk. Now the reads
will immediately go to the memory. It ignores the existence of disk(like database which is our persistent storage). If the
server crashes, all the data in memory would not be written to disk(persisted). To fix this, we periodically dump the data from memory to disk(copy
it). This is OK if you're fine with some amount of data loss. But for example, we wouldn't do this in twitter. For example liking and
disliking is not that important, so we can use this approach.

Note: We want to never read from cache(or reduce this as much as possible).

### Eviction policy
We need these policies because memory(cache) is limited in size.

- random
- FIFO: We're gonna remove the first one that was added and then add a new to end of it. But what if the one we're deleting is read a lot?
Like a tweet from popular celebrity. So we need other policies.
- LRU(least recently used): When a cache entry is read, we move it to the end, so it becomes the most recently used. So we won't delete it for now.
We want to remove the least recently used or read.
- LFU(least frequently used): we would have key-value pairs and the key maybe would be the tweet itself and then the value would be
the number of times that tweet was read. We remove the one that has the lowest value. So the most popular tweets are gonna stay in cache. But
we still allow new tweets to be added to cache and if they'll be read a lot, it won't be evicted.

In case of twitter we would prefer LRU. Because there could be really popular tweets but maybe it stopped. It was popular a tweet a year ago and
it reached high cache hits but after a while, it wasn't read a lot. With LFU, it would stay in our cache because of it's used count(value) since
it's so high. **So with LFU we don't care how recently it was read, we just care about the total count**.

In caching we wanna speed things up and we're willing to make some tradeoffs(like sacrificing consistency).

## 11-11 - CDNs
## 12-12 - Proxies and Load Balancing
## 13-13 - Consistent Hashing
## 14-14 - SQL
## 15-15 - NoSQL
## 16-16 - Replication and Sharding
## 17-17 - CAP Theorem
## 18-18 - Object Storage
## 19-19 - Message Queues