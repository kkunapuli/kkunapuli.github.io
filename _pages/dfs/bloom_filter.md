---
title:  "\'Bloom\'ing Efficient Filter"
layout: single
author_profile: true
header:
   image: /assets/images/jon-tyson-yeIERmyuTjc-unsplash_opt.jpg
   caption: "Photo by Jon Tyson on Unsplash"
---

> Bloom filters are named for their creator, Burton Howard Bloom. They have nothing to do with plants.

November 10th, 2019

A [bloom filter](https://en.wikipedia.org/wiki/Bloom_filter) is a probabilistic data structure. It's used to quickly exclude obviously irrelevant information in exchange for tolerating some false positives. A bloom filter can say "no, this item definitely doesn't belong", or "this item might belong, but it might not". The best part is that the false alarm positive can be calculated _and controlled_.

<img src="/assets/images/dfs/bloom_filter.png">
<figcaption> Bloom filter with multiple hash functions. </figcaption>


Bloom filters function a lot like a hash table, but they use memory much more efficiently. A bloom filter uses less memory because it doesn't deal with collisions, so it has no need to store the actual item. In fact, all we need is **one bit** to store  `true` or `false`. This means that a bloom filter can be computed once, stored in a compact bitmap, and loaded on startup or as needed. 

A hash table requires 4 or 8 **bytes** per index while a bloom filter only requires one **bit**.
{: .notice--warning}

To check if an item is in the desired set, simply hash it and check the corresponding bit in the bitmap. If the bit is toggled to `0`, there's no way this item is in the desired set. If the bit is set to `1`, this item _might_ be in the desired set or it _might_ be a collision, e.g. two items that hash to the same value. [Bloom Filters by Example](https://llimllib.github.io/bloomfilter-tutorial/) allows you to create and play with your own filter.

If you're familiar with [handling collisions](https://en.wikipedia.org/wiki/Hash_table#Collision_resolution) in a hash table, you're aware that multiple items (e.g. words) can hash to the same value. We try to avoid this by using a larger hash table, but some collisions will happen no matter what. In a bloom filter, we use the number of hash functions and the size of the filter to control the desired false positive rate. 

## Project Setup

The first step of a designing a good experiment is to identify a test case and declare an expected outcome. Let's find a large text file, and count a specific set of words. I would expect this to be faster than counting _all_ words.

I intentionally am not verifying set inclusion with this application. I **want** the false positives to be reported at the end; I'm going to measure the actual false positive rate and compare it to the rate predicted by the filter design. In practice the application would double check any bits set to 1 before reporting. 
{: .notice--warning}

I downloaded a text version of _Alice in Wonderland_ from [Project Gutenberg](https://www.gutenberg.org/) for testing. Then, I decided on a list of seven words to count: `alice, cake, cat, hatter, rabbit, queen, tea`. I threw in `computer` for fun, which brings the total to eight. 

We'll see some surprising words in the results (such as computer) because I left the Project Gutenberg license in the text flie.
{: .notice--warning}

### Generate Bloom Filters

Next, I set the **filter size based on a desired false positive rate**. This is an experiment, so I decided to make two filters: one with ~10% probability and one with ~1% probability of false positives. To keep things simple, I stuck with one hash function (a Java built-in). Probability of false positives, `P(FP)`, is _approximately_:

<a href="https://www.codecogs.com/eqnedit.php?latex=\LARGE&space;\mathbb{P}(FP)&space;=&space;(1-e^{\frac{kn}{m}})^{k}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\LARGE&space;\mathbb{P}(FP)&space;=&space;(1-e^{\frac{kn}{m}})^{k}" title="\LARGE \mathbb{P}(FP) = (1-e^{\frac{kn}{m}})^{k}" /></a>

- k hash functions used
- m total bits in the filter
- n items inserted in the filter (sometimes you'll have to guess 'n')

Using more hash functions allows you to use less memory without allowing more false postives, but it also requires more computations and can slow down the application. 
{: .notice--warning}

<img src="/assets/images/prob_FA.png">
<figcaption>Probability of False Positive vs. Bloom Filter Size (n=8). False positives are less frequent with larger filter sizes.</figcaption>

From the plot, if we want a 10% P(FP), we need a filter with 79 bits. We increase the filter size to 673 bits for a 1% P(FP). I had a few options for storing the filters on disk. I could have saved the raw bitmap in a binary file which would be very compact but not easily interpreted or verified. Instead, I saved only the index locations with a `1` or `true`, e.g. "hits". It's still compact, but it's much easier to inspect and understand.

```
Indices for 79 filter size:
[67, 75, 20, 18, 65, 53, 55, 67]

Indices for 673 filter size:
[101, 294, 619, 579, 4, 409, 611, 42]
```

We can already see the **effects of the probabilistic nature of our bloom filter**. The smaller filter (10% probability of false positives) has _two words map to the same filter index_. Since we populated our filter with eight words, we could expect 0.1 * 8 = 0.8 words to overlap (a probability of false alarm is actually the probability that unique words map to the same index). 

Notice, the number of bits in a filter isn't divisible by eight. Hash tables typically use a prime number for table size. We'll waste a few bytes, but it will be a small fraction of the total filter size in practice.
{: .notice--warning}

### Add Bloom Filter to Mapper

First, I created a Filter class leveraging Java's [BitSet](https://docs.oracle.com/javase/7/docs/api/java/util/BitSet.html) for the bitmap. Given a desired filter size, it creates a bitmap with the necessary number of bits. Then, it reads filter indices from disk and sets them to `1`.  

```java
import java.util.BitSet;
private static class Filter{
  BitSet filter;
  int filterSize = 0;
  Filter(String filterFile) throws NumberFormatException, IOException{
			
    //open filter file and create filter
    File file = new File(filterFile);
    BufferedReader br = new BufferedReader(new FileReader(file));
			
    // create bitmap
    filterSize = Integer.valueOf(br.readLine());
    filter = new BitSet(filterSize);
			
    // for each marked element in filter, update bitmap
    String s;
    while((s = br.readLine()) != null) {
      filter.set(Integer.valueOf(s));
    }
			
    br.close();
  }
		
  boolean isSet(int bit) {
    return filter.get(bit);
  }
		
  int getIndex(String word) {
    int index = word.hashCode()%filterSize;
    if(index < 0) {
      index += filterSize;
    }
			
    return index;
  }
}
```

Using an existing MapReduce [word count application](https://kkunapuli.github.io/_pages/dfs/map_reduce/) as a starting point, I added an optional bloom filter to the Mapper step. Our updated mapper looks like this:

```java
// print word + "1" for counting
public Mapper(String line, Filter filter) // filter may be null
{
  String[] tokens = line.split(" ");
  for(String temp: tokens)
  {
    temp = temp.toLowerCase();
			
    if(filter == null || filter.isSet(filter.getIndex(temp))){
      temp += " 1";
      stringList.add(temp);
    }
  }
}
```

## Measure Performance

### Generate a Baseline

First, let's do a full word count on _Alice in Wonderland_ without any code changes or filtering. This will establish a baseline for comparing runtimes and allow us to compute an accurate false positive rate (number of false positives divided by the total number of words). Using our existing [word counter](https://kkunapuli.github.io/_pages/dfs/map_reduce/):

```sh
$ time script/driver.sh alice_in_wonderland.txt 778 # count everything
real	0m0.719s
user	0m3.713s # amount of time spent by CPUs
sys	0m0.772s
```
It took our word counter about 3.7 seconds to tally ~3,000 unique qords. Some of the results indicate we could improve our filtering, but it's ok for now. Let's look at some of the results (remember, the Gutenberg License is in our text file).

```sh
$ sort -k 1 reduce_results.all.txt | tail -10
yet 25
you 481
young 5
your 71
yours 3
yourself 10
youth 6
zealand 1
zigzag 1
zip 1
```

### Test with ~10% Probability of False Positives

Using the smaller filter size, run the word counter again. Is it faster? How many extra words are reported?

```sh
$ time script/driver.sh alice_in_wonderland.txt 778 bloom_filter.79.txt 
real	0m0.634s
user	0m3.414s
sys	0m0.786s

$ cat reduce_results.79.txt | wc -l
     286 # this is about 10% of 3k unique words
```

A bit unexpectedly, the word counter with a filter only takes 0.3 seconds less CPU time. This is why it's a good idea to identify an application's bottlenecks _before optimizing it_. I suspect most of the processing time is spent in the Splitter, Stemmer, and Mapper and is unaffected by the filtering. Since the goal was to have some fun experimenting with bloom filters, I'm curious but not worried about timing results. 

## Test with ~ 1% Probability of False Positives

```sh
$ time script/driver.sh alice_in_wonderland.txt 778 bloom_filter.673.txt 
real	0m0.588s
user	0m3.324s
sys	0m0.747s

$ cat reduce_results.673.txt | wc -l
      36 # about 1% of 3k unique words
```
Using a larger filter, with smaller false positive probability, reduced timing a little more. More importantly, it greatly reduced the number of reported results.

All the words (filter words in **bold**):
* **alice 403**
* because 16
* binary 1
* books 2
* **cake 3**
* **cat 37**
* **computer 2**
* different 10
* directions 3
* draggled 1
* e 29
* end 20
* fallen 4
* fifteen 1
* fortunately 1
* **hatter 56**
* held 4
* hiss 1
* jaw 1
* kind 8
* kiss 1
* learned 1
* leave 9
* never 48
* order 3
* out 118
* **queen 75**
* **rabbit 51**
* say 51
* sends 1
* solid 1
* speed 1
* strange 5
* **tea 19**
* top 8
* water 5

In a production application, we would check for inclusion in our list of words before printing results (probably at the Reducer level). I skipped that step here so we could see what it really means to have a false positive rate. In the end, the filter reduced timing by 0.5 seconds or 13%. It's not a huge improvement, but then again it didn't take much effort to implement...

Because our set of words is so small, we'd be better off using a traditional set and having zero probability of false positives. The differnce in memory usage would be negligible and our output would only included the words of interest. Coding it up would also be simpler with a built-in set.






