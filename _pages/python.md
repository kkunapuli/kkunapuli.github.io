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
: Get a new copy of a list reversed with slicing:
```python
my_list = [0, 1, 2, 3, 4]
reverse_list = my_list[::-1]
print(my_list)
print(reverse_list)
```

List comprehension
: Create a list using an expression or by filtering an existing list
```python
{
odd_list = [i for i in my_list if i%2 == 1]
print(odd_list)
}
```

Iterate list with a value and index
: Ever want to iterate a list and feel stuck choosing between an iterator and range for loop?
