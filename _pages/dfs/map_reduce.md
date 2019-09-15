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

MapReduce has the best name: a MapReduce framework **maps** items to values and **reduces** the `(item, value)` pairs into answers. The trick is there's many Mappers operating at the same time on their own slice of data. From now on, we'll refer to Mapper pairs as `(key, value)` pairs [as per usual](https://docs.oracle.com/javase/tutorial/collections/interfaces/map.html).

**insert figure here**

MapReduce and [distributed file systems](https://kkunapuli.github.io/_pages/dfs/distributed_system/) go together like peanut butter and jelly. Because the Mappers perform independent calculations, their data products can be stored on separate machines in a distributed system.  
 
[MapReduce comes from Google](https://ai.google/research/pubs/pub62), but other technologies such as Spark and [Hadoop](https://www.tutorialspoint.com/hadoop/hadoop_mapreduce.htm) use a MapReduce framework too. Google has since developed [other technologies](https://ai.google/research/pubs/pub41378) that replaced MapReduce however it is still used in batch processing and there's plenty of value in learning MapReduce's simple approach to parallelism at scale.

Our application has five steps: Split, Stem, Map, Send, and Reduce. Splitter, Stemmer, Mapper and Sender communicate using pipes. Sender connects to Reducer using TCP. A bash script ties it all together and is the key to quickly and easily attaining parallel processing on a large scale. We'll go over all the steps, but most of the magic happens in the Reducer (where we collate results from disparate Mappers) and the bash driver. [GitHub project](https://github.com/kkunapuli/MapReduce).

## Split it.
If we're going to run multiple Mappers at the same time, we need to divide up the data - we don't want the Mappers to do duplicate work! There's a lot of approaches that will work, but a simple one is to simply divide the input file into fourths: one for each Mapper. 

You can run a MapReduce application with as many parallel processing chains as you like. We picked four.
{: .notice--warning}

Dividing up the file happens in the Splitter. The Splitter requires three command line arguments, input filename, number of lines in the file and a number 1-4 specifying which processing chain we're starting. Because it is written in java, our command line looks like this: `java -classpath bin/ Splitter $filename $num_lines $i`. After the Splitter calculates the range of line numbers, it prints those lines on **stdout**.

*Wait - I thought we're using pipes to communicate between processes?!?!* We are - Unix pipes!
{: .notice--warning}

Stemmer is up next. It reads strings from stdin, cleans it, and prints the result on stdout. Why clean the data? Because we're counting unique words. Consider the sentences _"Programming is great fun. Everyone should try it and see how much fun programming can be."_ The word "programming" appears both capitalized and lowercase; "fun" appears with and without trailing punctuation. We probably don't care how many times [stop words](https://en.wikipedia.org/wiki/Stop_words) such as "is", "it", & "and" appear. 

If you consider proper nouns, especially ones that aren't always proper such as "apple" the fruit and "Apple" the company, cleaning text can get more complicated. We're keeping it simple for now.
{: .notice--warning}

After Stemmer converts text to lowercase then removes stop words and punctuation including new lines. It too prints the result on stdout. 

### Relevant Files
- Splitter.java
- Stemmer.java

## Map it.
Next up is the Mapper; just like Stemmer, it receives input on stdin. Mapper's task is to convert input into `(key, value)` pairs. Because our application is counting words, the key is a word and the value is the current count. Why 1? Each item is a single word so its count is always 1. 

If our application was counting daily sales for each store by ID, our `(key, value)` pairs would be `(store ID, daily sales)`.
{: .notice--warning}

Mappers can also perform filtering. For example, we might only be interested in words beginning with "s" or how many times I can mention "data" in a single article. Mapper finishes by printing `$word,1` on stdout (separated by spaces). 

### Relevant Files
- Mapper.java

## Send it.

### Relevant Files
- Sender.java

## Reduce it.
There’s only one instance of the Reducer but four processing chains feed it information. In addition to processing input in a new thread, the Reducer must protect its intermediate results so that threads are not blocked and they don’t step on each other with a race condition. Without protecting the intermediate results, we could easily end up with multiple threads trying to add to the list of counts at the same time and end up with the wrong answer. 

The Reducer stores all intermediate results in a hash map where the key is the word and the value is a list of numbers (e.g. count). To avoid a race condition while allowing concurrency for maximum throughput, we synchronize the threads and protect the hashmap with a lock. 

```java
public boolean update(String key, int val) {
  //update word table with the key and value
  synchronized(wordLock){
    if(wordTable.containsKey(key)) {
	   //key exists; update entry
		wordTable.get(key).add(val);
    }
	 else {
      //key doesn't exist yet; add new entry
      ArrayList<Integer> tmpList = new ArrayList<Integer>();
      tmpList.add(val);
      wordTable.put(key, tmpList);
      }
    }
  return true;
}
```

All threads share one lock with minimal scope and no dependencies in order to prevent deadlock. Threads only need to acquire a lock when they are modifying the hash map, `wordTable`. 

### Relevant Files
- Reducer.java

## Drive it!
We tested our MapReduce application along component boundaries (e.g. Splitter, Stemmer, etc) but the real fun is running all components end-to-end.

Running and testing the MapReduce application is straight-forward when using the bash script driver.sh. The driver script starts the Reducer, then calls the Splitter-Stemmer-Mapper-Sender chain four times. All Reducer output is redirected to a file reduce_results.txt in the current working directory. Reducer output location can easily be changed by updating driver.sh. 

Here’s an example test scenario to demonstrate the MapReduce application. All steps assume you’re running from the MapReduce directory with script, and bin directories present.
```sh
$ pwd
 /Users/kkunapuli/projects/MapReduce
$ ls
 bin	script	src
```

1. Make a test file

	Using vim, we have a test file (lines.txt) that repeats “Here is a test line of text. Writing text is fun.” 400 times. It will test both stop word removal and accurate counting of words. Because “is”, “a”, and “of” are all stop words, we expect the final result to be 400 counts each of “here”, “test”, “line”, “writing”, and “fun”. Because “text” appears twice, it should have a count of 800. The text line will also test capitalization and punctuation removal.
	
		```sh
		$ cat lines.txt | head -5
		 Here is a test line of text. Writing text is fun.
		 Here is a test line of text. Writing text is fun.
		 Here is a test line of text. Writing text is fun.
		 Here is a test line of text. Writing text is fun.
		 Here is a test line of text. Writing text is fun.
		$ cat lines.txt | wc -l
     		 400
		```
		
	A couple reasons why I like vim:
		- it's available on even the most lightweight systems
		- sometimes I just want to quickly edit a file without waiting for a text editor to start
		- once you learn the [commands](https://vim.rtorr.com/), vim is incredibly powerful
	{: .notice--warning}
	
2. Run MapReduce application

	Run the entire MapReduce application with a single terminal command:
	
	`script/driver.sh $test_file $port`
	
	Use whatever (free) port number you like - we just need to make sure Sender and Reducer use the same one. Here, `$test_file` is lines.txt that we just created.
	
	I also added a Unix time command to demonstrate that our application is indeed using four threads of execution.
	
		```sh
		$ time script/driver.sh lines.txt 778
		 real	0m0.519s
		 user	0m2.050s
		 sys	0m0.787s
		```
		
	As we can see from the time output, our application took 519 ms of “wall clock time” to execute. More interesting is that it spent 2050 ms CPU time executing. That’s approximately 4 times longer than our wall clock estimate! *This is exactly what we would expect when starting four different (but equally loaded) processes to execute in parallel.* Our application did indeed use four parallel processes to count the number of unique word occurrences in lines.txt.
		
3. Inspect results

	All reducer output is directed to reduce_results.txt. It should contain a list of unique words in the input text file along with a count. In this case, we expect six unique words, text will have a count of 800, all others will have a count of 400.
	
		```sh
		$cat reduce_results.txt 
		 text 800
		 test 400
		 here 400
		 line 400
		 writing 400
		 fun 400
		```
		
	As you can see from the terminal output, “is”, “a”, and “of” were removed as stop words. The “.” was removed from text and it was matched correctly to the other instance of text in the second sentence. All remaining words were sent to the Mapper and Reducer and aggregated correctly. If there was an error in concurrency or synchronization, we would likely see repeated words with differing counts or incorrect counts.
	
	*Why aren't the words ordered?* Because we use a hashmap to store unique words and their current count. We achieve O(1) lookup but sacrifice any kind of ordering. Even if we listed words in the order they were received in the Reducer, we wouldn't be guaranteed the same order as the original file without significantly more synchronization overhead.
	{: .notice--warning}
	
4. Driver script anatomy

### Relevant Files
- script/driver.sh
