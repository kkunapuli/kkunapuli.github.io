---
title:  "python tips, tricks & how-to's"
layout: single
permalink: /python/
author_profile: true
header:
   image: /assets/images/randomWalk.jpg
---

Some useful pythonic idioms I've learned along with simply useful code snippets.

> ### Title Art
>
> Random Walk from [Patterns of the Universe](https://www.amazon.com/Patterns-Universe-Coloring-Adventure-Beauty/dp/1615193235). Given a grid of hexagons, assign a number and color to each of the six sides. Take a die and roll it to determine the next hexagon to fill in (and its color). Repeat. If you're wondering why there are more than six colors, good eye! I whimsically rotated colors in and out. **Colors by me**.

## Pythonic Idioms

Reverse a list
: Slicing will return a new list but reversed. Using method .reverse() will flip the list in place. 
```python
>>> # create a simple test list
... my_list = [0, 1, 2, 3 , 4, 5]
>>> 
>>> # get a new reversed list using slicing
... reverse_list = my_list[::-1]
>>> my_list
[0, 1, 2, 3, 4, 5]
>>> reverse_list
[5, 4, 3, 2, 1, 0]
>>> 
>>> # or, reverse in place using .reverse()
... my_list.reverse()
>>> my_list
[5, 4, 3, 2, 1, 0]
```

List comprehension
: Easily create a new custom list or filter an existing one with list comprehensions. 
```python
>>> # create a list with odd values between 1 and 10
... odd_list = [i for i in range(10) if i%2==1]
>>> odd_list
[1, 3, 5, 7, 9]
>>> 
>>> # square odd_list
... square_list = [i*i for i in odd_list]
>>> square_list
[1, 9, 25, 49, 81]
```
The general format is  `[expression for item in list if condition]`

Iterate list with a value and index
: Python makes it really easy to iterate through a list, but sometimes I *still* need an index. Luckily `enumerate()` returns both index and value.
```python
>>> # create a list of strings
... fruits = ['apple', 'orange', 'banana', 'mango']
>>> # iterate through list 
... for idx, ifruit in enumerate(fruits):
...     print("fruits[{}] = {}".format(idx, ifruit))
... 
fruits[0] = apple
fruits[1] = orange
fruits[2] = banana
fruits[3] = mango
```
When using a dictionary, use `dict.items()` instead of `enumerate()` to loop over key, value pairs.
