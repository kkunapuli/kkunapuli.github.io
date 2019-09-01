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

To ensure synchronous communication, Name Node creates a thread of type 'Name Node Handler' for each Client request. The Name Node Handler then creates its own connection to Data Nodes - who are always listening on a specific port number. Name Node creates a new connection every time it wants to communicate with a Data Node.

### Relevant Files
- Client.java
- NameNode.java

## Name Node
Name Node is the middle man of the DFS as it communicates with both Clients and Data Nodes. Its core job is to maintain a record of which blocks to request from a Data Node if, or when, a Client makes a request. Because Name Node can field multiple simultaneous client requests - using Name Node Handler threads - the map containing a record of blocks on Data Nodes is protected by a mutex. 

When receiving an APPEND request, Name Node first splits the data into 4 MB blocks then contacts a Data Node to request a free block. If a block is available, Data Node returns the block number and marks the block as used. Name Node then sends a WRITE request to the same Data Node. Name Node employs a Round Robin algorithm to decide which Data Node to use. Name Node simply rotates through its list of Data Nodes, requesting one block at a time, until all data has been stored.

### Relevant Files:
- NameNode.java
- Pair.java

## Data Node
Data Node serves as the storeroom of the DFS. It has two primary data structures: a queue of available blocks and a hashmap of used blocks. The queue and hashmap have their own individual locks (qLock for queue and uLock for hashmap); whenever a data structure is modified, Data Node first obtains the relevant lock, modifies the data structure and then releases the lock.

All locks have a minimal scope in order to prevent deadlock or excessive blocking.
{: .notice--primary}

Because we are simulating a DFS on a single computer, Data Node saves a block of data in its own directory: data_<port>. Each allocated block of data is represented by a file: blk_<num>.bin; Data Node does not put a limit on block filesize (Name Node handles 4MB limit).

Data Node accepts a port number as a command line argument and starts a ServerSocket on the specified port. It then listens continuously on its port and accepts a client connection from Name Node when a new message comes in. Each new message is handled in a new thread: Data Node Handler. 

It is important to immediately start a new thread when a message comes in so that subsequent incoming messages are not blocked.
{: .notice--primary}

Data Node Handler extends Java’s Thread class and its constructor takes two arguments: the message as a String, and a DataNode object. The reference to Data Node is given as an argument to its thread so that Data Node Handler can perform alloc(), read(), and write() requests on Data Node. With using locks, a data node has to manage its own resources; thus a handler has access to a single instance of a data node (representing a unique host). If data can be modified in one place it is simpler to ensure its safety.

**insert new figure here**
The Communication process between NameNode and one DataNode is shown in “DataNode Message Handling” figure. Although only one DataNode is shown, in reality the NameNode connects to and communicates with all DataNodes.

Concurrency is a huge conern for Data Nodes. We can ensure that data is kept safe by performing all actions sequentially, but then requests would take much longer than if processed in parallel. See [reader writer problem](https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem) for more information. Each allocated block has a file name, read lock, and write lock (using [Java's ReadWriteLock class](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/ReentrantReadWriteLock.html)).  In order to allow a read on block x1 while writing to block x2 of the same DataNode, each block has its own instance of ReentrantReadWriteLock. For simplicity, a Block object has a read lock (rLock) and a write lock (wLock). 

Both locks are declared as final so that they cannot be modified. To emphasize this point, they were not given ‘setter’ methods in the Block class. {:. notice--primary}

We get writer preference for free with Java’s ReadWriteLock, i.e. if a DataNodeHandler is writing to block x1 or requests write access to x1, no readers will be given access to x1. Readers may be given access to any Block other than x1 since Blocks have independent ReadWriteLocks. We also get the ability to allow multiple readers on the same block at the same time from ReadWriteLock.

### Relevant Files:
- StartDataNodes.java
- DataNode.java
- DataNodeHandler.java
- Block.java

## Running and Testing DFS
This is the fun part; we get to see the DFS in action! 

Creates multiple DataNode processes on unique ports. Each process is independent and does not communicate with the other DataNodes. This means that they manage their own data and have their locks. We chose not to share memory/data structures between DataNode processes because in a physical system, a DataNode would likely be independent computer with no knowledge of any other nodes.Stderr and stdout messages are directed to a log file named DNode_<port>.log.
