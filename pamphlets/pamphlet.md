## 0 - Computer Architecture
Disk = storage = hard drive(HDD hard disk drive, nowadays SSD is more common)

Memory = RAM(random access memory)

Reading and writing(done by CPU) from RAM is measured in terms of microseconds(1/10^6) and it's a lot faster than reading and
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