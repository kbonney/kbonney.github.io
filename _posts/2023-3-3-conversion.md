---
title: "Converting MATLAB to Python: The Saga Begins"
flavor: "The Good, The Bad, and The 'Damn, I really should have taken more CS courses in college'"
tag: "blog"
---

This post is a launching point for a series of investigations I plan to make on the process
of converting between two scientific computing languages: MATLAB and Python

## The Good
There are many reasons to want to convert matlab code to python code:
- lack of a license
- personal preference
- the need to access libraries in python without equivalents in matlab

Whatever your reason, you definitely don't want to this by hand. Despite the similarities between the languages, any sizeable amount of matlab code will take a signficant effort to bring over to python.

For those who stumbled upon this page only looking for the best tool to perform automatic conversion, I will point you to [matlab2python](https://github.com/ebranlard/matlab2python). This is simply the only functioning attempt at matlab to python translation on the web right now. The repository is still receiving active development, with latest commits coming in a couple of months ago. There is no shortage of other projects which have attempted some sort of conversion, but they are all long since deprecated. 

I was able to use matlab2python right out of the box, and got great results within a few minutes. However, as the authors mention in their README, you should not expect anything to work perfectly right out of the box. In my experience, the conversion gets you anywhere from 50% to 90% of the way there, depending greatly on the complexity of the task and how nitpicky you are on the style of the code. The one piece of advice I will offer for making use of the outputs is to get really familiar with regex. It will save you a lot of time and pain. 

With that out of the way, I want to turn to the main topic:

## The Bad
I used matlab2python (ml2py) to bring a very large, very legacy code base into the Python language. At this point, ml2py has probably converted over 10,000 lines of code for me and for that I am eternally grateful. However, throughout that experience I noticed a number of issues with the conversion process that felt very fixable, and some issues that felt less fixable - but very interesting. To elaborate on this, we'll take a look an example conversion.

Before I continue, I want to stress that none of this is meant as a criticism of the work done by the ml2py maintainers or the backend from [smop](https://github.com/victorlei/smop/) that they build upon. Rather, this is meant to illustrate the nuances of the conversion process and to motivate us to learn more about what is going on behind the scenes.

Now, let's jump into an example matlab script `ml_script.m`:
```
N = [10,11,12,13];
total = 0;
for i = 1:4
	total += N(i);
end
```
Pretty basic script, right? You would hope that this would be a very simple case to take care of - a nice clean output that runs with no bugs. Somehow, this is not the case. Running ml2py on this file gives us the output `py_script.py`:
```
import numpy as np
N = np.array([10,11,12,13])
total = 0
for i in np.arange(1,4+1).reshape(-1):
	total += N(i)
```
This code fails to run. This code looks bad. Why?

The code fails to run for two reasons:
1. `N` is a numpy array, which must be accessed using square brackets. Parentheses are reserved for calling in Python, which is not implemented for arrays.
2. In the for loop, `i` starts at 1 and ends at 4. However, unlike MATLAB, python indexing starts at 0. The maximum index of `N` is at 3, so `N[i]` when `i==4` will raise an exception.
It may seem strange that we would fail on such a simple scenario, but there are good justifications for this. 

Matlab and Python are both dynamically typed languages. Nowhere in either the matlab script nor the python output do we explicitly assign data types to our variables. Without a very complex backend, there is no way for a conversion process to determine whether a variable name is a function, class, floating point number, array, or anything else. The simple solution is to not try to do this, which as far as I can tell, is what causes error 1. 

Our converter doesn't realize that the name `N` is actually pointing to a matrix. On the matlab side this is okay because matrices, functions, and classes all use parentheses - so it doesn't really matter. However, when we try to bring things to Python it does matter - matrices need square brackets while functions and classes still use parentheses.  

To address error 2, it might seem like all we have to do is replace instances of `X:Y` in matlab with `np.arange(Y-1,X).reshape(-1)` in Python rather than `np.arange(Y,X+1).reshape(-1)`. In general, there are multiple things that would break if we were to do this. I want to focus on one that is particularly interesting. Consider the following matlab script:
```
for i=1:5
	for j=1:5
		disp(i*j)
	end
end
```
Our proposed conversion would take this to
```
for i in np.arange(0,5):
	for j in np.arange(0,5):
		print(i*j)
```
These functions will not print out the same results. In fact it is worse. The results will not even have a nice one-to-one mapping. The first five lines from the matlab script are
```
1
2
3
4
5
```
but for the Python script they are
```
0
0
0
0
0
```
When we do this proposed conversion for `X:Y` we will ruin any arithmetic done on the iterators. 

## The 'Damn, I really should have taken more CS courses in college'
Motivated by the types of issues I just outlined, I started reading through the ml2py codebase to see if I could find a way to remedy my problems with a helpful pull request. As I read the code, and did more research on relevant topics like **parsers**, **lexers**, **abstract syntax trees**, and whatnot, I realized I found the whole process of conversion fascinating. 

What I plan to do in a series of posts is explore the process of computational language conversion. At the risk of reinventing the wheel, I want to try my hand at building a matlab to python conversion from scratch. I don't intend to produce something as comprehensive as what can be found in ml2py, but I do want to attempt to take a closer look at the two problems I laid out in a more rigorous context. 

1. What kind of machinery is necessary to distinguish matrices from functions in matlab?
2. How can we account for the indexing mismatch between the languages, without causing more problems than we solve?

This will require diving into the topics I mentioned earlier. We'll look at some theory, we'll look at some code, we'll go off the rails. I think it will be fun.