---
bg: "wine.jpg"
layout: post
title:  "Introducing Object Everywhere!"
summary: "Why storage systems work better on object"
categories: posts
tags: 'overview'
---
This post looks at Object Storage from an "IT organization" perspective, such as the CTO. As a CTO, the requirements of a storage system are to provide the users of the system something that will meet their diverse needs, while also maximizing the total cost of ownership of the system itself. The CTO is responsible for keeping the data of the organization safe and has to balance the difficulty of managing ever growing piles of data locally with the unknowns of moving data and applications into the cloud. For storage systems specifically there is a permanence that means changing technologies is not just a re-writing of applications, but a migration potentially involving months of work, outages for applications and large costs. It is not a decision that is taken lightly.

Lets start with what a definition of a storage system. A storage system converts storage hardware, typically with a block interface, to a useable interface like a POSIX file system, an S3 interface or a SQL interface. A storage system is primarily responsible for providing availability and durability to application data at a speed that the application needs. It must also scale as the application demands and provide multi-tenant access and security of the data. Finally it needs to do this at a reasonable cost. 

The hierarchical file system was and is the most common interface today, but its popularity is waning as it struggles to meet these needs for new applications.

Object storage first came on the scene about 12 years ago as a reaction to the limitations of traditional file systems. It has been around in various flavors for years, but there was no unified standard, and few compelling reasons to use it. Cleversafe erasure coded object storage helped change that by targeting customers who were crippled by the weight of file storage. On the cloud, it was popularized by [Amazon S3](https://press.aboutamazon.com/news-releases/news-release-details/amazon-web-services-launches-amazon-s3-simple-storage-service) in 2016 as the basis for their public cloud and their implementation has become the de facto standard. What makes Object Storage so attractive today from a systems perspective compared to other storage technologies is primarily five factors:

* *Scale* - single namespace deployments in the hundreds of petabytes to exabytes
* *Performance* - due to its simple interface - it grows linearly as the system size grows
* *Cost* - typically 1/4 the cost of comparable file system alternative
* *Availability* - designed with virtually perfect durability and availability
* *Security* - built in Internet age and uses modern security techniques

Hierarchical file systems were first introduced over [50 years ago](https://www.multicians.org/fjcc4.html) and were designed in an era where computers were direct attached to their storage and capacities were measured in MB. POSIX helped standardize the interface in the late 1980's and network attached file systems and scale out systems soon appeared. However some of the original assumptions and decisions are no longer valid and have resulted in a lot of complexity and performance trade-offs file systems. 

Object storage is unshackled by legacy POSIX file systems requirements and a way to think of it is "File systems for the Internet". 

Scale
-----

Scale is the primary reason that is cited for using object storage, but it can be confusing why this is the case looking just at file system specs. As an example, ext4 claims 1EB scale with billions of files, and XFS supports 8EB volumes with 2^64 files however  looking at the [RedHat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/performance_tuning_guide/chap-red_hat_enterprise_linux-performance_tuning_guide-storage_and_file_systems#sect-Red_Hat_Enterprise_Linux-Performance_Tuning_Guide-Considerations-File_systems) on this, they only support a maximum size of 50TB for ext4 and 500TB for XFS. With [Seagate now at 16TB](https://blog.seagate.com/craftsman-ship/hamr-milestone-seagate-achieves-16tb-capacity-on-internal-hamr-test-units/) spinning drives shortly and [Nimbus shipping 100TB SSD](https://nimbusdata.com/products/exadrive-platform/scalable-ssds/), it is no longer even possible to mount an ext4 file system over a single drive (let alone a RAID volume of these drives). 

It is worth examining why a file system "theoretical" capacity is so much lower than the claimed number. First is the repair time - a PB scale ext4 system would take [hours to days](https://www.enterprisestorageforum.com/storage-hardware/the-state-of-file-systems-technology-problem-statement.html) to recover from corruption. Journaling has helped make this less necessary, but it is still not possible

Performance
-----------

Performance is something that everyone likes to tout but is complex to measure. If you think about it from an end user perspective, what they really care about is "how long do I have to wait to get what I want done" which I'll call **Wait time**. This can be waiting for a web page to load or a complex analytics job to complete. Unfortunately when you look at a performance system you typically see **IOPS** - how many I/O operations can I complete per second. And it is virtually impossible to convert from "IOPS -> Wait time". When comparing "similar" systems (e.g. two file systems) - it is sometimes possible to use IOPS as a proxy for Wait time, but only when the interface is the same. Comparing a file system IOPS to a block or object IOPS number is meaningless, as they work at different layers. 

Other ways of comparing performance are operation latency or system throughput, however again these are a poor proxy for **Wait time**. So what is a user to do to try and decide how they want to compare what there system performance will look like? 

* First, decide how you want to measure performance and if the storage system is even your bottleneck, if not this shouldn't be part of your selection criteria
* Second, measure or find measurements online about how your application works against different storage systems
* Third, make sure you optimize for the right metrics

An object storage system can have throughput orders of magnitude higher than a file system at high scale. We have built systems with millions of spinning drives that can sustain 1TB/s and millions of operations per second.


Cost
----
From a customer perspective, cost is up with performance on one of the most important metrics for a storage buyer. It may seem like an interface decision should not have a cost implication, but in reality it makes a huge difference. Holding all else constant, an object storage systems is typically [4-5x less](https://www.prnewswire.com/news-releases/cleversafe-and-shutterfly-working-together-to-solve-evolving-storage-challenges-128781543.html) expensive than a comparable file storage system. The cost savings come from multiple sources. 
* Storage redundancy - object storage systems typically use erasure coding instead of replication for data durability (half as much hardware)
* Commodity hardware - object storage is typically on commodity hardware rather than proprietary file hardware and redundancy is in software rather than hardware (half the cost)
* Much lower maintenance cost - due to the redundancy and commodity hardware, the servicing needs are greatly reduced as well.

Putting these pieces together the cost of an object storage system is much closer to the "bare media cost" than a file system.


Availability
------------
For many applications, being able to run 24/7 without maintenance windows is imperative. With file systems this is typically difficult as there is a "head node" that controls access. More recently scale out file systems have worked around this limitation, but typically file systems are more difficult to "scale up" rather than "scale out". This is intrinsically due to the centralized natures of file system. An object storage system typically treats each object as independent and if built with redundancy or erasure coding has the ability to use consensus to achieve availability even in the face of node outages (either planned or unplanned). 

Data durability is another area where object storage systems can make a large difference. 

Security
--------
Security can be modeled using the [CIA triad](https://en.wikipedia.org/wiki/Information_security) - Confidentiality, Integrity and Availability. File systems were not designed with any security in mind. Initial systems were directly attached and single user. POSIX added minimal support for multi-user access with users and groups, but they were meant for a small set of "trusted users". There have been numerous enhancements to improve the situation, but it is difficult to bolt on security without changing the fundamental API of POSIX. Network security is not part of the NFS spec, and there is no notion of encryption at rest built into the APIs. There are ways to make NFS more secure, but it is very rare to see an NFS connection (or any other file interface) outside of a VPN. [Recent efforts](https://blogs.oracle.com/linux/encrypting-nfs-data-on-the-wire) have attempted to bring better security to NFS, but it is unclear if they will gain widespread acceptance. Object storage on the other-hand had the benefit of treating security as a first-class citizen from the start. From a Confidentiality perspective, all transfers are done over a TLS connection. Additionally the API supports adding encryption either by the end user, or by the storage system using a key of the users choice. Integrity is maintained by transferring a checksum along with the data on both reads and writes. Finally, Availability in object storage is achieved by serving data over the HTTP interface. There are various appliances and tools that prevent attacks such as DDoS against an Object system. The security built into Object makes it a "first class" internet protocol that is often deployed as an "open to the Internet" protocol.
