---
title:  "Split it. Map it. Send it. Reduce it!"
layout: single
author_profile: true
header:
   image: /assets/images/marilia-castelli-l74VaCZon7I-unsplash_opt_opt.jpg
   caption: "Photo by Marília Castelli on Unsplash"
---

>When I think of MapReduce, I picture an army of robots shuffling numbers.

September, 2019

MapReduce has the best name: a MapReduce framework **maps** items to values and **reduces** the `(item, value)` pairs into answers. The trick is there's many Mappers operating at the same time on their own slice of data. **insert figure here**

MapReduce and [distributed file systems](https://kkunapuli.github.io/_pages/dfs/distributed_system/) go together like peanut butter and jelly. Because the Mappers perform independent calculations, their data products can be stored on separate machines in a distributed system.  
 
[MapReduce comes from Google](https://ai.google/research/pubs/pub62), but other technologies such as Spark and [Hadoop](https://www.tutorialspoint.com/hadoop/hadoop_mapreduce.htm) use a MapReduce framework too.Google has since developed [other technologies](https://ai.google/research/pubs/pub41378) that replaced MapReduce but it is still used in batch processing and there's plenty of value in learning MapReduce's simple approach to parallelism at scale.

Our application has five steps: Split, Stem, Map, Send, and Reduce. A bash script ties it all together and is the key to quickly and easily attaining parallel processing on a large scale. We'll go over all the steps, but most of the magic happens in the Reducer (where we collate results from disparate Mappers) and the bash driver. [GitHub project](https://github.com/kkunapuli/MapReduce).

## Split it.
include information on stemmer since there's literally nothing to either of these

### Relevant Files
- Splitter.java
- Stemmer.java

## Map it.

### Relevant Files
- Mapper.java

## Send it.

### Relevant Files
- Sender.java

## Reduce it.

### Relevant Files
- Reducer.java

## Drive it!

### Relevant Files
- script/driver.sh
