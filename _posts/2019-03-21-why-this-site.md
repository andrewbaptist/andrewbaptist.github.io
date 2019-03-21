---
bg: "wine.jpg"
layout: post
title:  "Why Object Storage"
summary: "What is object storage, and why should I care about it?"
categories: posts
tags: 'overview'
---
I started this site to answer the question "Why Object Storage?". There are various dimensions to this question and I hope to answer many of them. 

* Why would a user of a storage system write to an Object Storage interface rather than a traditional file system or database?
* Why would an IT organization want to present Object Storage to my end users?
* What does the architecture of a distributed storage system look like?
* What is the future of storage systems?

I have been developing Object Storage systems for 11 years, and have seen the rapid changes in that time. I expect the next 11 years will be even more dramatic driven by the continued explosion of data generating devices and the need to generate meaning from that data. The underlying storage hardware technology will continue to rapidly progress from HDD to SSD to Persistent Memory, and the interfaces users use to connect with these devices will change with it. 

Storage technology has changed dramatically over the past 30 years as devices have connected to the Internet and moved data from "paper" to "electronic bits". Today the size of storage devices is mind-bogglingly large. Individual hard drives and SSD drives are 16TB or more (that is 16,000,000,000,000 bytes) and the largest storage systems, including ones built on IBM COS technology, are over 1 EB (1,000,000,000,000,000,000). It is difficult to even fathom the scale of these systems. In the near future, we will be talking about systems considerably larger with much more demanding performance implications.

On the other hand, many developers are only starting to use object technologies and IT divisions are only starting to understand why they would want object storage in their environments. There is confusion about what applications are appropriate for object storage vs traditional storage and what advantages (and disadvantages) switching over would entail. There are questions about how new applications should be written and how to maintain legacy applications that pre-dated object storage. 

I want to try and answer many of these questions in this blog. I hope to write a new entry every 1-2 weeks and will tailor them towards helping answer these questions.
