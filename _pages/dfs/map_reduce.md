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

MapReduce has the best name: a MapReduce framework *maps* items to values and *reduces* them into answers. The trick is that there's many Mappers operating at the same time on their own slice of data. **insert figure here**

[MapReduce comes from Google](https://ai.google/research/pubs/pub62), but other technologies such as [Hadoop](https://www.tutorialspoint.com/hadoop/hadoop_mapreduce.htm) use a MapReduce framework too. MapReduce and [distributed file systems](https://kkunapuli.github.io/_pages/dfs/distributed_system/) are like peanut butter and jelly; because the Mappers perform independent calculations, their data products can be stored on separate machines in a distributed system. Google has since developed [other technologies](https://ai.google/research/pubs/pub41378) that replaced MapReduce but it is still used in batch processing and there's plenty of value in learning MapReduce's simple approach to parallelism at scale.

Our application has five steps: Split, Stem, Map, Send, and Reduce. A bash script ties it all together and is the key to attaining parallel processing on a large scale quickly and easily. We'll go over all the steps, but most of the magic happens in the Reducer (where we collate results from disparate Mappers) and the bash driver.

## Split it.
include information on stemmer since there's literally nothing to either of these

## Map it.

## Send it.

## Reduce it.

## Drive it!
