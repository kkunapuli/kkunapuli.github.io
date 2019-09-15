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
 
[MapReduce comes from Google](https://ai.google/research/pubs/pub62), but other technologies such as Spark and [Hadoop](https://www.tutorialspoint.com/hadoop/hadoop_mapreduce.htm) use a MapReduce framework too. Google has since developed [other technologies](https://ai.google/research/pubs/pub41378) that replaced MapReduce however it is still used in batch processing and there's plenty of value in learning MapReduce's simple approach to parallelism at scale.

Our application has five steps: Split, Stem, Map, Send, and Reduce. Splitter, Stemmer, Mapper and Sender communicate using pipes. Sender connects to Reducer using TCP. A bash script ties it all together and is the key to quickly and easily attaining parallel processing on a large scale. We'll go over all the steps, but most of the magic happens in the Reducer (where we collate results from disparate Mappers) and the bash driver. [GitHub project](https://github.com/kkunapuli/MapReduce).

## Split it.
If we're going to run multiple Mappers at the same time, we need to divide up the data - we don't want the Mappers to do duplicate work! There's a lot of approaches that will work, but a simple one is to simply divide the input file into fourths: one for each Mapper. 

You can run a MapReduce application with as many parallel processing chains as you like. We picked four.
{: .notice--warning}

Dividing up the file happens in the Splitter. The Splitter requires three command line arguments, input filename, number of lines in the file and a number 1-4 specifying which processing chain we're starting. Because it's java, our command line looks like this: `java -classpath bin/ Splitter $filename $num_lines $i`. After the Splitter calculates the range of line numbers, it prints those lines on **stdout**.

_Wait - I thought we're using pipes to communicate between processes?!?!_ We are - Unix pipes!
{: .notice-warning}

Stemmer is up next. It reads strings from stdin, cleans it, and prints the result on stdout. Why clean the data? We're counting unique words. Consider the sentences _"Writing is great fun. Everyone should try it out and see how much fun writing can be."_ The word "writing" appears both capitalized and lowercase; "fun" appears with and without trailing punctuation. We probably don't care how many times [stop words](https://en.wikipedia.org/wiki/Stop_words) such as "is", "it", "and" appear. 

If you consider proper nouns, especially ones that aren't always proper such as "apple" the fruit and "Apple" the company, cleaning text can get more complicated. We're keeping it simple for now.
{: .notice-warning}

After Stemmer converts text to lowercase then removes stop words and punctuation including new lines. It too prints the result on stdout. 

### Relevant Files
- Splitter.java
- Stemmer.java

## Map it.
Next up is the Mapper; just like Stemmer, it receives input on stdin. Mapper's task is to convert input into `(key, value)` pairs. Because our application is counting words, the key is a word and the value is the current count. Why 1? Each item is a single word so its count is always 1. 

If our application was counting daily sales for each store by ID, our `(key, value)` pairs would be `(store ID, daily sales)`.
{: .notice-warning}

Mappers can also perform filtering. For example, we might only be interested in words beginning with "s" or how many times I can mention "data" in a single article. Mapper finishes by printing `$word,1` on stdout (separated by spaces). 

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
