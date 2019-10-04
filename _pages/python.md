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
The general format is  `[`EXPRESSION `for` ITEM `in` LIST `if` CONDITION`]`

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

Swap items in a single line
: Python allows 'multiple assignment' where you can assign n values to n variables; multiple assignment feature also allows us to swap two or more items in a single line of code.
```python
>>> # create two variables, with initial values using multiple assignment
>>> a, b = 2, 3 # a=2, b=3
>>> a
2
>>> b
3
>>> # swap values of a and b using multiple assignment again
>>> a, b = b, a
>>> a
3
>>> b
2
>>>
>>> # extends to N items and can be used with other objects like lists
>>> a, b, c = [0, 1, 2], [3, 4, 5], [6, 7, 8]
>>> print('a={}, b={}, c={}'.format(a, b, c))
a=[0, 1, 2], b=[3, 4, 5], c=[6, 7, 8]
>>> # rotate a, b, c lists (a=b, b=c, c=a)
>>> print('a={}, b={}, c={}'.format(a, b, c))
a=[3, 4, 5], b=[6, 7, 8], c=[0, 1, 2]
```

Default values when accessing dictionaries
: A common use for dictionaries is to store a count for each word (or other item). When we want to include another word in the running counts, we need to first check if the word is in the dictionary to avoid errors, right? We could. Or we could use `.get()` with a default value!
```python
>>> word_list = ['mango', 'banana', 'plum', 'pear', 'plum', 'apple', 'mango', 'plum', 'pear']
>>> word_counts = {} # initialize empty dictionary
>>> for w in word_list:
...     word_counts[w] = word_counts.get(w, 0) + 1 # if w not in dictionary, return 0
... 
>>> word_counts
{'mango': 2, 'banana': 1, 'plum': 3, 'pear': 2, 'apple': 1}
```

## Python Data Structures

Modify a character in a string
: Strings are immutable (i.e. can't modify) in Python, but that doesn't mean we're stuck. Python makes it easy to convert a string to a list and back again.
```python
>>> # as an example, let's use a word ladder to convert 'lost' to 'coal'
>>> str = 'lost'
>>> str_list = list(str)
>>> str_list
['l', 'o', 's', 't']
>>> str_list[0] = 'c' # turn 'lost' into 'cost'
>>> str_list
['c', 'o', 's', 't']
>>> str_list[2:] = ['a', 'l'] # do two steps in one to turn 'cost' into 'coal'
>>> str_list
['c', 'o', 'a', 'l']
>>> str = "".join(str_list) # convert list back into a string
>>> str
'coal'
```

Use Python's built-in heap
:  Did you know Python has built-in support for [heaps](https://en.wikipedia.org/wiki/Heap_(data_structure)) (aka priority queues)? It's a great way to maintain relative order if, for example, you need to find the smallest item repeatedly. If you're used to storing trees or heaps in an array, you won't be surprised to learn that you can achieve heap functionality with a python list with `heapq`. Warning, the heapq methods modify your list.
```python
>>> my_list = [15, 20, 8, 6, 4, 12, 3, 9]
>>> heapq.heapify(my_list) # put the list in approximate order (based on heap rules)
>>> my_list
[3, 4, 8, 6, 20, 12, 15, 9]
>>> heapq.heappush(my_list, 1) # add some items to the heap
>>> heapq.heappush(my_list, 2)
>>> my_list
[1, 2, 8, 4, 3, 12, 15, 9, 6, 20] # order is different!
>>> print('smallest item in heap = {}'.format(heapq.heappop(my_list)))
smallest item in heap = 1
>>> print('next 3 smallest items are: {}'.format(heapq.nsmallest(3, my_list)))
next 3 smallest items are: [2, 3, 4]
```

Merge two lists
: When adding a single item to a list, you can use `.append()`. But if you need to add an entire _list_, use `.extend()`.
```python
>>> list1 = [0, 1, 2]
>>> list2 = [3, 4, 5]
>>> list1.extend(list2)
>>> list1
[0, 1, 2, 3, 4, 5]
```

Track insertion order in a dictionary with `OrderedDict`
: Dictionaries (a.k.a. hashmaps) are awesome and very convenient, but the keys aren't guaranteed to be in a particular order. Python has `OrderedDict` in collections that will keep keys in _insertion order_, in exchange for extra memory of course. [Documentation here](https://docs.python.org/3/library/collections.html#collections.OrderedDict).
```python
>>> from collections import OrderedDict
>>> ordered = OrderedDict()
>>> ordered.update({0:0, 1:1, 2:2, 3:3}) # merge two dictionaries with .update()
>>> ordered.keys()
odict_keys([0, 1, 2, 3])
>>> ordered[1] = 3
>>> ordered.keys()
odict_keys([0, 1, 2, 3]) # order hasn't changed
>>> ordered.move_to_end(1)
>>> ordered.keys()
odict_keys([0, 2, 3, 1]) # 1 is now at the end
```

