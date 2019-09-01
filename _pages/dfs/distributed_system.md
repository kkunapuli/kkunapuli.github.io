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
There are three types of actors: Client, Name Node, and Data Node. For this project we have one Name Node that communicates with multiple Data Nodes and Clients. Data Nodes store information or "data" in fixed blocks of size 4 MB. 

<img src="/assets/images/dfs/dfg_main_opt.png">
<figcaption>General Distributed File System with N Data Nodes. While only one client is shown, multiple clients can access the system at a time.</figcaption>


Clients send requests to Name Node; they can either APPEND data to a file or READ from a file. For APPEND commands, Name Node finds free blocks on one or more Data Nodes for storage. With READ commands, Name collects applicable blocks from Data Nodes and assembles them in order to return to the Client.

It's important to utilize threads to ensure that Name Node and Data Nodes can handle multiple requests. Each incoming request is processed as a new thread. Data Nodes have additional data protection mechanisms in place that parallel access without issue.

## Communication Design
Name Node needs to be able to communicate with both clients and data nodes seamlessly. Thus, Name Node is a server to Clients and a client to Data Nodes. Processes share data through ports over TCP.

To ensure synchronous communication, Name Node creates a thread of type 'NameNodeHandler' for each Client request. The NameNodeHandler then creates its own connection to Data Nodes - who are always listening on a specific port number. Name Node creates a new connection every time it wants to communicate with a Data Node.

### Relevant Files
- Client.java
- NameNode.java

## Name Node
Name Node is the middle man of the DFS as it communicates with both Clients and Data Nodes. Its core job is to maintain a record of which blocks to request from a Data Node if, or when, a Client makes a request. Because Name Node can field multiple simultaneous client requests - using NameNodeHandler threads - the map containing a record of blocks on Data Nodes is protected by a mutex. 

When receiving an APPEND request, Name Node first plits the data into 4 MB blocks then contacts a Data Node to request a free block. If a block is available, Data Node returns the block number and marks the block as used. Name Node then sends a WRITE request to the same Data Node. Name Node employs a Round Robin algorithm to decide which Data Node to use. Name Node simply rotates through its list of Data Nodes, requesting one block at a time, until all data has been stored.

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

