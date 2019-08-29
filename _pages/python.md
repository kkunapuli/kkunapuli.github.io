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
>>> a
[0, 1, 2]
>>> b
[3, 4, 5]
>>> c
[6, 7, 8]
>>> # rotate a, b, c lists (a=b, b=c, c=a)
>>> a, b, c = b, c, a
>>> a
[3, 4, 5]
>>> b
[6, 7, 8]
>>> c
[0, 1, 2]
```

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
