---
title:  "data & machine learning"
layout: single
permalink: /sandbox/
author_profile: true
header:
   image: /assets/images/primes.jpg
---

Things I've had fun learning and exploring.

> ### Title Art
>
> Ulam Spiral from [Patterns of the Universe](https://www.amazon.com/Patterns-Universe-Coloring-Adventure-Beauty/dp/1615193235). Starting with 1 in the center, each square in a 20x20 grid is numbered in a counter-clockwise spiral. I chose to color all primes red, then even numbers true blue, ones divisible by 3 avocado green, and so on. **Colors by me**.

## Big Data

[Store ALL the Data]({% link _pages/dfs/distributed_system.md %})
: I had a great time designing and building a Distributed File System (DFS) with a team and wanted to share some of the things I learned.

[Split it. Map it. Send it. Count it!]({% link _pages/dfs/map_reduce.md %})
:  With a team, I explored concurrency and synchonization by creating a MapReduce application. It's a simple use case - count the number of occurrences of unique words - but requires muilti-threading, locks, and a barrier to get it right.

['Bloom'ing Efficient Filter]({% link _pages/dfs/bloom_filter.md %})
: What if we want to count a small subset of words instead of _all_ words? What if our small subset is still a **lot** of words? Bloom filters are commonly used for quick filtering with a reduced memory footprint.

## Resources
[Tools for Adding Content]({% link _pages/resources/website.md %})
: Everything I found helpful in building and adding to this site.
