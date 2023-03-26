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
## 5-5 - DNS
## 6-6 - HTTP
## 7-7 - WebSockets
## 8-8 - API Paradigms
## 9-9 - API Design
## 10-10 - Caching
## 11-11 - CDNs
## 12-12 - Proxies and Load Balancing
## 13-13 - Consistent Hashing
## 14-14 - SQL
## 15-15 - NoSQL
## 16-16 - Replication and Sharding
## 17-17 - CAP Theorem
## 18-18 - Object Storage
## 19-19 - Message Queues