---
date: '2025-07-02T08:48:01+09:00'
draft: false
title: 'Level0'
cover: 
    image: img/calc.jpg
    alt: 'This is a post image'
    caption: 'calc.jpg'
tags: ["html","css"]
categories: ["PythonChallenge"]
---

What is Python Challenge ?

Python challenge is a set of puzzles that can be solved by using Python.

Currently there are 33 levels.

IMHO Python Challenge is not for beginners who have never coded in Python before or have little experience in Python.

It's for people who have some experience in Python but just aren't super good.

If it's your first time writing Python code I wouldn't try to solve these problems.
Reading O'reilly books or learning from online courses would help you more.

For courses I recommend:
{{< youtube xAcTmDO6NTI >}}

The course explains Python in a pretty detailed manner for beginners.


You should try to read as many Python books as possible. Python Challenge is for people who know Python but just aren't familiar with using various modules and writing classes in Python.

Each level in Python Challenge usually requires you to use a certain module.

## Level 0

The title for level0 is warming up.
It's always a good idea to check the HTML title because the HTML titles are usually helpful to solve the problems.

By looking at the tab where you opened the pythonchallenge problem, you can check the title.

![title](/img/pythonchallenge/level0/warming_up.png)

The title of level0 is warming_up.

After checking the HTML title you should also have a look at the image file Python Challenge provides you.

Usually the image files title are also hint to solve the puzzle. You can view that using right-click and Inspect.
You'll then see some HTML tags.

The tag after `<img src>` is the name of the file.
In level 0 `<img src="calc.jpg">` it's calc.jpg.

From the title I can guess that this level shouldn't be that hard.

Level0 has a picture of an old desktop PC 

{{< figure src="/img/calc.jpg" alt="desktop" width="640" height="480" >}}


On the monitor there is a yellow rectangle with `2**38` written on it.


Below the image of the computer there is a hint that tells you to change the URL address.


With all the information given it looks like I should use `2**38` and change the URL.


The current URL is http://www.pythonchallenge.com/pc/def/0.html.


If you look closely you can see that the html file starts with a number.


Calculating `2**38` and replacing it from 0 would probably solve level0.


Python has many ways to calculate `2**38`.


1\. Using the `**` operator

You can directly use \*\* to calculate powers of numbers.

```python
print(2**38)
```


2\. Using the bit shift operator


Python provides bit shifting operators. That would be another option.


```python
print(2 << 37)
```


3\. Using the `pow` function
Python provides the built-in [pow](https://docs.python.org/3/library/functions.html#pow) to calculate powers of numbers.


The pow function also let's you calculate the modular as well.


```python
print(pow(2, 38))
```


`2**38` gives us **`274877906944`**.


Changing the URL from [0.html](http://www.pythonchallenge.com/pc/def/0.html) to [274877906944.html](http://www.pythonchallenge.com/pc/def/0.html) takes you to level1.


I'll try to explain the code and how to solve these problems in detail as much as possible.


I struggled a lot when solving these problems on my own.


The solutions or writeups I found online were hard to understand.


That might be because I'm dumb or something lol but anyways...