---
bg: "julia.png"
layout: post
title:  "Designing scalability"
summary: "Ways to build infinitely large storage systems"
categories: posts
tags: 'overview'
---

In a perfectly scalable storage system, there are a few attributes you would like:
* The scale of the storage system should be larger than anything you could conceivably use
* It should be possible to start with a small system and grow it over time. 
* The performance of the system should scale linearly as the system grows, both in operations per second and total throughput. 

Most storage systems, both file and object, are not designed this way as these are hard goals to obtain.

This post covers how the IBM COS system is built to meet these goals. This approach can be applied to other object storage systems due to the "atomicity" of objects on two levels.
* *Objects are independent of each other* - there is no notion of a directory which contains a set of objects
* *Objects are fully overwritten* - from a readers point of view it instantly flips from the old to the new object

This approach does not apply as directly to file systems since they do not have either of these properties, however as Ceph has shown it is possible to build a file system on top of an object storage ssytem.

Designing an object storage system that can store a practically infinite amount of storage, both in number of objects and capacity, requires different solutions. Approaches that may be logarithmic as the data set size grows don't work as well when there are trillions of objects. It is no longer acceptable to have logarithmic time lookup of data or centralized anything. It sounds almost impossible to design a solution that can do this. Typical file systems can store millions to billions of objects. Cleversafe wanted to build a system that could store many many more. 

Points - compare to COS Ceph, OpenStack swift, Scality Ring

There are two key architectural pieces that scalable object storage systems employ
* Map the objects to a number using a hashing function
* Map the namespace to nodes using a mapping function

The first part (hashing) is easy to think about. Take the name of an object, and convert it to a "big number" using a hashing function. A simple (but effective) example would be to compute the SHA-256("object name") as its value. A good hashing function (like SHA) has the property that small changes in the input are very far apart in the output, and that it is very difficult/unlikely that two names will have the same hash value. The SHA-256 function computes a 256 bit value, so there are 2^256 possible different values in can compute. This is a mind-blowingly large number (more than the number of atoms in the universe) - and therefore even taking into account the birthday paradox, it is exceedingly unlikely that a natural collision of two names will ever occur in any system.

The second part is a little more complicated, however at a high level it is easy to do. It relies on the property that the hashing function will generate a name with a uniform distribution (equally likely to be any name in the system). The simplest approach is to simply assign a range of numbers to each nodes. For instance if there were 1000 possible names (0-999) and 10 nodes, assign the first 100 names to the first node, the second 100 to the second node and so on.  There are a few additional considerations to deal with this in production which I will discuss solutions to these below, however the high level approach works well. 

With this approach, lets run through a practical example. In this example I'll use hex numbers instead of binary for simplicity.

16 nodes - each owns the 

Object Name = 'foo1234'
Hash Value = sha256('foo1234') = 'aad17a81b5ffb3e8f4907f988efc5e99091525f30ecca2f41c6678787d74ad07' 
Mapping from hash to node = -> node 10 (a000000000000000000... - affffffffffff)

There are a couple considerations that need to be consider to make this approach work 
* Not all nodes are the same size
* We want "multiple homes" for an object for durability reasons

For different size nodes, there is a straighforward solution. At the time of system creation, assign a weight to the node according to its size. If a node is twice as big as another, then it stores twice the number of hash values. This simple approach works well as long as the node values at creation are static. This solution appears to be fairly consistent across most systems that use namespace mapping.

For "multiple homes" for an object, there are a few different approaches and depending on the system, and other guarantees that are made regarding consistency of reads/writes different approaches can be taken. I will discuss a two different alternatives, but others are possible. 

Option 1 - use different hashing functions for each of the pieces. For instance compute hashes of "foo1234:0", "foo1234:1" and "foo1234:2" and then store to these three locations or how many different locations are necessary. The drawback to this approach is that there is no guarantee that the different values land on different nodes no matter how many nodes we have. Probabilistically, the data will fall on different nodes, but some data will end up with hash values very close together and end up next to each other.

Option 2 - compute a single hash and rotate the computed value by 1/NAMES_TO_COMPUTE.  The benefit of using the "rotate" approach is that we guarantee that the names are maximally distant which is an important property when we optimize for failures.  The first thing that needs to happen is to compute the location of the multiple homes. A simple way to do this is to rotate around the "circle" by the number of names you want to compute. Taking the example above, if we want a second name for the same hash, just add MAX/2 to the value (this is an unsigned int, so it wraps around). If we want 3 names, the simply add MAX/3 and 2\*MAX/3 to the value and we now have 3 names. If we want 24 names, add MAX/24, 2\*MAX/24, ... 23\*MAX/24 to the name and you have all the names necessary. Now each of these names can be mapped to the nodes that hold it. A nice property of this approach is that different objects can have a different number of homes. For object that are transient and don't require any additional durability, simply have 1 home for the object. For an object that needs 3 copies, compute 3 homes. If using erasure coding, compute homes for each of the erasure coded segments. 


With these top two approaches we have a practical toy system that would work well in the lab. In practice there are two more considerations 
* We want to be able to grow/shrink the system over time
* Not all nodes are up all the time

To deal with the system changing size, there are a few different approaches that can be taken, all of them have tradeoffs, none of them are perfect, and different object storage solutions have chosen different options. I won't go into the details on all the solutions, but will on the one I think is best (Weighted Rendevous Hashing)
* Don't allow changing the system size. This is the most similar to a block or file system that doesn't allow resizing and may be acceptable for some uses, but is probably not ideal
* Treat the systems as distinct. When writing chose one "semi-randomly", when reading attempt to read from all. This works for a very small number of different sets
* Remap all the data when the size changes. This requires a massive data migration as data will "waterfall" from one node to the next. Adding a single node at the end requires almost half the data in the system to move.
* Group the nodes into sets, use a mapping function to chose the set first, then chose the node within that set.

In case for a static system that is always up, however there are additional considerations when the system changes size. There are a few different approaches to dealing with this. The simplest approach if we add a new node to the system is to apply a remapping function and move the data from its current location to the "new location" so that 

