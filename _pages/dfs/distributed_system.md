---
title:  "Store ALL the Data"
layout: single
author_profile: true
header:
   image: /assets/images/pranav-kumar-jain-PR_0IPlMXgk-unsplash_opt_crop.jpg
   caption: "Photo by Pranav Kumar Jain on Unsplash"
---

June, 2019

Distributed File Systems (DFS) allow us to store and access huge amounts of data as if it was available locally. Sometimes a server runs out of space or it's simply not feasible to expect that all relevant data be stored on a single machine. With a distributed file system, we can easily add more space (and compute capacity) without impacting clients. Sounds simple enough but if we're going to successfully handle multiple requests from multiple clients, we need to have **conncurrency** and **synchronization** mechanisms in place. 

**My core contribution to the project was the Data Node**; it will be the focus of this discussion. [GitHub project](https://github.com/kkunapuli/DistributedFileSystem)

## Basic Architecture
For this project we have one Name Node that communicates with multiple Data Nodes and Clients. 
<img src="/images/dfs/dfg_main_opt.png">
<figcaption>General Distributed File System with N Data Nodes<figcaption>

## Communication Design
Name Node needs to be able to communicate with both clients and data nodes seamlessly. 

### Relevant Files:
- Client.java
- NameNode.java

## Name Node
Name Node is the middle man of the DFS as it communicates with both Clients and Data Nodes. It's core job is to maintain a record of where data is stored on Data Node blocks if, or when, a Client makes a request.

### Relevant Files:
- NameNode.java
- Pair.java

## Data Node
Data Node serves as the storeroom of the DFS. 

### Relevant Files:
- StartDataNodes.java
- DataNode.java
- DataNodeHandler.java
- Block.java

## Running and Testing DFS
This is the fun part; we get to see the DFS in action! 

