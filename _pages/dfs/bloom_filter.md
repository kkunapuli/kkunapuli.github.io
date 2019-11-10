---
title:  "\'Bloom\'ing Fast Filter"
layout: single
author_profile: true
header:
   image: /assets/images/jon-tyson-yeIERmyuTjc-unsplash_opt.jpg
   caption: "Photo by Jon Tyson on Unsplash"
---

> Bloom filters are named for their creator, Burton Howard Bloom. They have nothing to do with plants.

November 10th, 2019

## Bloom Filter Background
A [bloom filter](https://en.wikipedia.org/wiki/Bloom_filter) is a probabilistic data structure. It's used to quickly exclude obviously irrelevant information in exchange for tolerating some false positives. A bloom filter can say "no, this item definitely doesn't belong", or "this item might belong, but I could be wrong". The best part is that the false alarm positive can be calculated _and controlled_.

**insert image here**

Bloom filters function a lot like a hash table, but they use memory much more efficiently. A bloom filter uses less memory because it doesn't deal with collisions, so it has no need to store the actual item. In fact, all we need is **one bit** to store  `true` or `false`. This means that a bloom filter can be computed once, stored in a compact bitmap, and loaded on startup or as needed. 

To check if an item is in the desired set, simply hash it and check the corresponding bit in the bitmap. If the bit is toggled to 0, there's no way this item is in the desired set. If the bit is set to 1, this item _might_ be in the desired set or it _might_ be a collision, e.g. two items that hash to the same value. [Bloom Filters by Example](https://llimllib.github.io/bloomfilter-tutorial/) allows you to create and play with your own filter.

If you're familiar with [handling collisions](https://en.wikipedia.org/wiki/Hash_table#Collision_resolution) in a hash table, you're aware that multiple items (e.g. words) can hash to the same value. We try to avoid this by using a larger hash table, but some collisions will happen no matter what. In a bloom filter, we use the number of hash functions and the size of the filter to control the desired false positive rate. 

## Project Setup

The first step of a designing a good experiment is to identify a test case and declare an expected outcome. Let's find a large text file, and count a specific set of words. I would expect this to be faster than counting _all_ words.

I intentionally am not verifying set inclusion with this application. I **want** the false positives to be reported at the end; I'm going to measure the actual false positive rate and compare it to the rate predicted by the filter design. In practice the application would double check any bits set to 1 before reporting. 
{: .notice--warning}

I downloaded a text version of _Alice in Wonderland_ from [Project Gutenberg](https://www.gutenberg.org/) for testing. Then, I decided on a list of eight words to count: `alice, cake, cat, hatter, rabbit, queen, tea`. I threw in `computer` for fun. 

- finding and setting a false alarm rate, see image
<img src="/assets/images/prob_FA.png">
<figcaption>Probability of False Positive vs. Bloom Filter Size. False positives are less frequent with larger filter sizes.</figcaption>

- create some filters and write out just the "hits" (sparse compression)
`Indices for 79 filter size:
[67, 75, 20, 18, 65, 53, 55, 67]
Indices for 673 filter size:
[101, 294, 619, 579, 4, 409, 611, 42]`

- memory usage for these?


- using Java's [BitSet](https://docs.oracle.com/javase/7/docs/api/java/util/BitSet.html)

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

now our mapper looks like this:
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

## Getting a Baseline
run alice in wonderland without any changes
$ time script/driver.sh alice_in_wonderland.txt 778 # count everything
real	0m0.719s
user	0m3.713s
sys	0m0.772s

Some results: (~3k unique words)
Some numbers in there that indicate we could improve our filtering
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

## Try with ~10% Probability of False Positive
```sh
$ time script/driver.sh alice_in_wonderland.txt 778 bloom_filter.79.txt 
real	0m0.634s
user	0m3.414s
sys	0m0.786s

$ cat reduce_results.79.txt | wc -l
     286 # this is about 10% of 3k unique words
```

## Try with ~ 1% Probability of False Positive
```sh
$ time script/driver.sh alice_in_wonderland.txt 778 bloom_filter.673.txt 
real	0m0.588s
user	0m3.324s
sys	0m0.747s

$ cat reduce_results.673.txt | wc -l
      36 # about 1% of 3k unique words
```

All the words:
**alice 403**
because 16
binary 1
books 2
**cake 3**
**cat 37**
**computer 2**
different 10
directions 3
draggled 1
e 29
end 20
fallen 4
fifteen 1
fortunately 1
**hatter 56**
held 4
hiss 1
jaw 1
kind 8
kiss 1
learned 1
leave 9
never 48
order 3
out 118
**queen 75**
**rabbit 51**
say 51
sends 1
solid 1
speed 1
strange 5
**tea 19**
top 8
water 5

## So, what does this all mean?
We would actually check for inclusion in our list of words before printing results (probably at the reducer) but I let it all go through so we could see what it really means to have a false positive rate

Didn't save that much time...

In reality, we would have a much larger set to check against; we'd be better off using a traditional set and having P(FP) = 0.







