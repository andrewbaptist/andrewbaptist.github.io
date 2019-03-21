---
layout: post
title:  "Comparing Object to Traditional Storage Interfaces"
summary: "Contrasting object and file interfaces"
categories: posts
tags: 'overview'
published: true
---

A common question for anyone considering object storage is "why would I want to switch?". The first post dealt with the differences between a file and object system architecture and described why object storage systems can be bigger, faster and more secure. But this post looks at it from the "users perspective" to see why they would prefer object storage over file storage, even if they don't worry about those benefits. 

For a developer a file system is likely the first I/O system you deal with. Every programming language has first tier library support for a file interface. But the library support for file is more indicative of the deficiencies of the file interface rather than some intrinsic strength.

A developer who wants to talk to "store data" typically has to think first about whether that data is going to memory or disk. Typically there are two reasons to write things to disk 

* Needs to be non-volatile 
* More data than will fit in RAM

However from an interface perspective, there is not necessarily a reason to treat this differently.

If someone is storing a collection of data in memory, they will typically have a key for each piece of data and a value they store. An interface for a map in a typically programming language looks like:

<pre>
put(name, value)
value get(name)
delete(name)
Iterator(name, value) iterate()
</pre>

On the other hand a file system looks nothing like this. It has a notion of files, and each file needs to exist within a directory. Directories have strange limitations about naming, size, depth and number of items per directory. These are different on different file systems, but no file system will support billions of files in a single directory (and even if it some file system did, most don't so trying to write portable code to a file system requires respecting some of these limits). Additionally a file system is a "operating system notion" so there are differences between different operating systems. 

[Eat My Data: How everybody gets file I/O wrong](http://www.flamingspork.com/talks/2007/06/eat_my_data.odp)

Finally implementing something like "put" is incredibly difficult for a file system. [Atomically rewrite the content of a file](http://www.microhowto.info/howto/atomically_rewrite_the_content_of_a_file.html). Most developers do this simple operation (replace file contents) wrong that when ext4 was originally released for linux, it broke a vast number of applications and after widespread complaints and derision, the default was changed back to allow these applications to work, even though it meant the file system itself was slower. 

Finally durability with a file system is very difficult to achieve in any type of performant manner. Operations like fsync exist to allow a developer to force data to be written to disk, but as soon as the application and the file system are remote (because of something like NFS) - it becomes difficult to know if that fsync will work or not, and when it needs to be issued.
![File access](/assets/images/fileStack.png)

Looking at a picture of what a file stack is (rightmost picture) helps explain why this is so complex to get right. There are multiple layers (OS File System, NFS Client, Network, Gateway Network, NFS Server, Server stack) that all have different policies on sync and caching, and it is hard to know when your data is finally on media and no longer volatile. 

Lets compare this with an object interface. Here is a simple intro to using the [S3 API:](https://www.baeldung.com/aws-s3-java). Once you get past some of the boilerplate of installing, and setting up a bucket, the S3 operations look eerily similar to the Map interface I defined above
<pre>
putObject(bucketName, name, localFile)
S3Object getObject(bucketName, name)
deleteObject(bucketName, name)
ObjectListing listObject(bucketName)
</pre>
The operations a are scoped with a bucket because this is a "global" space for reading/writing objects, but this model allows a 1-1 translation of data from a map structure in RAM to a persistent structure.

// TODO: SQL interface
// TODO: latency

