---
layout: post
title: "Python Basics For Programmers"
date: 2014-09-14 07:10:17 -0400
author: Biju Nair
comments: true
categories: [python, programming]
---
Once we understand the common social norms of interacting with others, it is a matter of knowing how to express it in the particular language to the person we are interacting with. Similarly once we understand the fundamentals of computer programming, it is a matter of knowing the syntax of a particular language to express what needs to be accomplished. The following details the basic syntax of the Python language which will help get started with it and be able to build solutions for most of the programming tasks. The assumption is that the reader is familiar with fundamentals of programming and have been programming is another language.

<!-- more -->

Python programs are exectued through an interpreter called `python` and falls under the category of interpreted languages i.e. small chunks of code gets executed by an intrepreter program. So it is a pre-req to install the python interpreter. Python programs can be stored in a file with the extension `.py` normally referred as python script and passed as input to the python interpreter or python code can be executed through the python interpreter command prompt.

{% codeblock %}
python helloworld.py
Hello World!!
{% endcodeblock %}

Or the python interpreter can be invoked by the `python` command. Code can be developed and tested in the python interpreter command line.

{% codeblock %}
HW10478:python $ python
Python 2.7.2 (default, Oct 11 2012, 20:14:37)
[GCC 4.2.1 Compatible Apple Clang 4.0 (tags/Apple/clang-418.0.60)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> a = "Hello World"
>>> print a
Hello World
>>>      
{% endcodeblock %}

<h3> Variables </h3>
Python is a dynamically typed language i.e. the type of the variables are determined during runtime based on what is stored and doesn't have to be specified when variables are defined. Variable names can be of arbitrary length, can contain letter, numbers and underscore. But variable name need to start with a letter and avoid reserved words (no surprises!!). We can find the type of the variable using the `type()` function.
{% codeblock %}
>>> a = "Hello World"
>>> type(a)
<type 'str'>
>>> a = 10
>>> type(a)
<type 'int'>
{% endcodeblock %}

<h3> Operators and Expressions </h3>
All the arithmetic operators `` + - * / % ** `` are supported by Python and follows PEMDAS (Parentheses, Exponentiation, Multiplication, Division, Addition, Subtraction) order of execution. Python 3 has a new operator ` // ` which performs floor division i.e. ` 5/6 ` will produce a flot value of ` 0.8333333333333334 ` while it is zero in Python 2 and ` 5//6 ` will result in zero in Python 3. The ` + * ` operators can be used on String data as in the following examples

{% codeblock %}
>>> a = 'Hello '
>>> a * 3
'Hello Hello Hello '
>>> a + 'World'
'Hello World'
{% endcodeblock %}

Also all the standard relational operators ` == != < <= > >= ` and logical operators ` and or not ` are supported. Basic Python expressions are same as in any other language. 

<h3> Data Structures </h3>
Python has inbuilt ` list, tuple, dictionary ` data structures. 
**Lists** are similar to arrays where the indexes are integers and are auto assigned. Lists can store objects of mixed types and they are mutable. Some of the list functions are ` append ` `extend` `insert(idx,val)` `pop` `count` `sort` `reverse`

{% codeblock %} 
>>> a = [1,2,3]
>>> a.append([4,5])
>>> a
[1, 2, 3, [4, 5]]
>>> a.extend([6,7])
>>> a
[1, 2, 3, [4, 5], 6, 7]
>>> a.reverse()
>>> a
[7, 6, [4, 5], 3, 2, 1]
>>> a.sort()
>>> a
[1, 2, 3, 6, 7, [4, 5]]
>>> a.pop()
[4, 5]
>>> a[1]
2
>>> a[2:]
[3, 6, 7]
>>> a[:2]
[1, 2]
>>> a[0:2]
[1, 2]
>>> del a[0]
>>> a
[2, 3, 6, 7]
{% endcodeblock %}

**Tuples** are similar to lists but are not mutable and are created by using `( )` instead of `[]` used for lists. All the functions which can be used to access elements in list can be used to access elements of tuples. Since tuples are not mutable functions which modifies list like ` append pop del extend ` are not supported by tuples.

{% codeblock %}
>>> b = (1,2,3)
>>> b[1:]
(2, 3)
>>> b.count(3)
1
>>> b[:2]
(1, 2)
>>>
{% endcodeblock %}

**Dictionaries** are unordered set of key value pairs and indexed by the keys. ` { } ` are used to create dictionaries and they are mutable. Elements in dictionaries are accessed using the key values. All datatypes including tuples which are not mutable can be used as key values for dictionaries.

{% codeblock %}
>>> a =  {'a':1,'b':2,'c':3,'d':4}
>>> a
{'a': 1, 'c': 3, 'b': 2, 'd': 4}
>>> a['c']
3
>>> a['e']=5
>>> a
{'a': 1, 'c': 3, 'b': 2, 'e': 5, 'd': 4}
>>> a[('f','g')]=56
>>> a
{'a': 1, 'c': 3, 'b': 2, 'e': 5, 'd': 4, ('f', 'g'): 56}
>>> del a['a']
>>> a
{'c': 3, 'b': 2, 'e': 5, 'd': 4, ('f', 'g'): 56}
{% endcodeblock %}

` len ` is a common function across all the three data structures which can be used to find the length i.e the number of elements stored in them.

{% codeblock %}
>>> len(a)
5
{% endcodeblock %}

All the three data structures have iterators which can be used to iterate through all the elements 

{% codeblock %}
>>> a
{'c': 3, 'b': 2, 'e': 5, 'd': 4, ('f', 'g'): 56}
>>> for i in a:
...   print i, a[i]
...
c 3
b 2
e 5
d 4
('f', 'g') 56
{% endcodeblock %}

If you have a list, a tuple can be created using the ` tuple ` function

{% codeblock %}
>>> a = [1,2,3]
>>> c = tuple(a)
>>> c
(1, 2, 3)
{% endcodeblock %}

If you have a tuple, a list can be created using the ` list ` function.

{% codeblock %}
>>> d = list(c)
>>> d
[1, 2, 3]
{% endcodeblock %}

If you have two lists then a list of tuples can be created using the ` zip ` function.

{% codeblock %}
>>> a = [1,2,3]
>>> d
[1, 2, 3]
>>> f = zip(a,d)
>>> f
[(1, 1), (2, 2), (3, 3)]
{% endcodeblock %}

When you have a list of tuples as in the previous example, then a dictionary can be created using the ` dict ` function.

{% codeblock %}
>>> g = dict(f)
>>> g
{1: 1, 2: 2, 3: 3}
{% endcodeblock %}

Also when you have a dictionary a list of tuples can be created using the ` items ` dictionary function.

{% codeblock %}
>>> h = g.items()
>>> h
[(1, 1), (2, 2), (3, 3)]
{% endcodeblock %} 

Note that **strings** are nothing but list of characters. 

<h3> Functions and Modules </h3>
Sequence of related statements can be grouped in to a function so that it can be reused. Group of related functions can be saved in a file and it is called a **module**. For e.g. Pythons inbuilt module **math** stores mathematical functions like ` sqrt cos sin ` etc. To access a function in a module, the module need to be imported into the python script or in the interpreter command line. If you are someone who know **C** programing, this will be very familiar.

{% codeblock %}
>>> import math
>>> math.sqrt(25)
5.0
{% endcodeblock %}

When the whole module is imported as above, then a specific function need to be called by qualifying it with the module name like in the code above ` math.sqrt() `. If you are interested only in a certain function is a module, that particular module can be imported as in the following example.

{% codeblock %}
>>> from math import sqrt
>>> sqrt(36)
6.0
{% endcodeblock %}

Note that when a specific function is imported as above, it doesn't have to be qualified with the module name when the function is called.

<h4> Defining a function </h4>
Functions can be defined using a ` def ` keyword ` def function_name(optional parameters): `. The statements which belong to the function is grouped by intendation using spaces. The end of the function is determined by a blank space after the function or begining of another function or end of the file storing the function. 

{% codeblock %}
def farr(arrin):
  a = arrin
  print a
{% endcodeblock %}

All parameters and variables defined in a function are all local to the function. Any changes made to the parameters passed to the function will not be reflected in the parent code calling the function i.e. all the parameters are passed by value and not by reference.

When a group of related functions are stored in a file then it becomes a module and the file name is the module name. As with the inbuilt module like ` math `, functions from user defined modules can be made available for use in python scripts using the ` import ` statement.

<h3> Comments </h3>
Documenting the code is key. ` # ` can be used to add comments to Python code and anything added after ` # ` until the end of line is considered comment. Also Python supports adding documentationusing ` docstring ` to modules and functions so that they can read by users using the ` __doc__ ` function. ` docstring ` starts and ends with ` """ ` *triple quotes*. 

{% codeblock %}
>>> def coolfunc():
...   """ This is a cool function which does nothing !!! """
...   print "Cool function"
...
>>> coolfunc.__doc__
' This is a cool function which does nothing !!! '
>>>
{% endcodeblock %}

<h3> Execution Flow Control </h3>
Commonly found ` if ` ` for ` ` while ` statements are supported in Python. As in function definition, ` : ` is used to define the start of flow control  and the list of statments to be executed is grouped using proper intendation.

{% codeblock %}
>>> a = [1,2,3]
>>> for i in a:
...   print i
...   if i > 2:
...     print "Got you!!"
...
1
2
3
Got you!!
{% endcodeblock %}

Python **for** statement is different from other languages where users can define an initial value, end condition and an arithmetic expression which can be used to control the number of times the loop gets executed. In python `for` statement iterates over the items of any sequence in the order they appear in the sequence. In the previous example the for statement iterates over the list "a". ` range ` function can create a number list which inturn can be used in ` for ` loops.

{% codeblock %}
>>> for i in range(1,3):
...    print i
...
1
2
{% endcodeblock %}

` if ` statement supports ` elif ` which prevents deep code indendation and ` else ` clauses.

{% codeblock %}
>>> n = 10
>>> if n < 5:
...   print 'Less than 5'
... elif n >= 5:
...   print 'Greater than 5'
... else:
...   print 'Unknown'
...
Greater than 5
{% endcodeblock %} 

` while ` statement is similar to other languages.

{% codeblock %}
>>> n = 3
>>> while n < 5:
...   print n
...   n += 1
...
3
4
{% endcodeblock %}

` break ` and ` continue ` statements are available to control the ` for ` and ` while ` loops. 

<h3> Console Input Output </h3>

` raw_input() ` function can be used to receive input from users through **console**. The function takes an optional String, which will be output on console and waits for user to input value.

{% codeblock %}
>>> x = raw_input()
Test
>>> x
'Test'
>>> x = raw_input("Enter value :")
Enter value :Test
>>> x
'Test'
>>>
{% endcodeblock %}

Output to **console** can be done using ` print ` statement which takes in a string. If values need to be passed dynamically to the string to be displayed they can be done using the ` % ` operator.

{% codeblock %}
>>> print 'Test %s %s' % ('Hello','world')
Test Hello world
>>> print 'Test %s %s. Test Count %d' % ('Hello', 'world!', 2)
Test Hello world!. Test Count 2
{% endcodeblock %}

Note that the values passed should be elements if a tuple.

<h3> Files </h3>

` open ` and ` close ` functions can be used to (as you guessed) open and close files. ` open ` function takes in the file name including the path to the file and a character to indicate whether the file need to be opened for read or write. The returned value of ` open ` is a file handle which can be used for file processing and at the end of processing can be used to close the file.

` readline ` function can be used to read a line from the file at a time and ` read ` can be used to read the complete file. ` write ` function can be used to write data into the file.

{% codeblock %}
>>> fout = open('data.txt','w')
>>> fout.write('This is test data 1')
>>> fout.write('This is test data 2')
>>> fout.close()
>>> fin = open('data.txt','r')
>>> data = fin.readline()
>>> data
'This is test data 1This is test data 2'
>>> fin.close()
>>>
{% endcodeblock %}

<h3> Exception Handling </h3>

Similar to "try" "catch" block in JAVA for exception handling, Python uses ` try ` ` except ` clauses to handle exceptions.

{% codeblock %}
try:
  fout = open('invalid_file_name.txt','r')
except:
  print 'File open failed'
{% endcodeblock %}

<h3> Executing Python Scripts </h3>

As you may have noticed all the code block shown so far is using the Python command line interpreter. If a Python script is created and stored in a file then the script can be executed using the ` python filename ` command.

For e.g., if the following is the contend of the Python script file test.py
{% codeblock %}
def double(i):
    i += 1
    return i

x = 1
x = double(x)
print 'Value of x is %d' % x
{% endcodeblock %}

Then the script can be executed as below
{% codeblock %}
> python test.py
Value of x is 2
{% endcodeblock %} 

If a file (module) is created with Python functions, the functions can be exected as in the following example. This will avoid the functions to be executed when the module is imported into another script.

{% codeblock %}
> cat modu.py
def test1():
   print "Test 1"

def test2():
   print "Test 2"

if __name__ == "__main__" :
   test1()
   test2()
{% endcodeblock %}

{% codeblock %}
> python modu.py
Test 1
Test 2
{% endcodeblock %} 

What is covered so far could be used to accomplish most of the fundamantal tasks. This is intended to give a quick start on Python so that scripts can be created and used by the reader. 
