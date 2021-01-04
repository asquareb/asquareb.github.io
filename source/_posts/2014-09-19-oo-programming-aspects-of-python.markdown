---
layout: post
title: "OO Programming aspects of Python"
date: 2014-09-19 16:36:31 -0400
author: Biju Nair
comments: true
categories: [python, programming]
---
For anyone familiar with Object Oriented programming this gives a quick run down on the OO aspects of Python language.

A **class** can be created using the ` class ` keyword.

{% codeblock %}
>>> class Shape(object):
...     """This documents the details about the class Shape"""
...
>>> Shape.__doc__
'This documents the details about the class Shape'
{% endcodeblock %}

<!-- more -->

` object ` in the class definition specifies that the class Point inherits from the inbuilt class "object". Some other OO languages like JAVA this is implicit and doesn't have to be mentioned during class definition. So if a class need to inherit from another class then "object" can be replaced with the required class name.

{% codeblock %}
>>> class Square(Shape):
...     """ Square is a shape """
...
>>> Square.__doc__
' Square is a shape '
{% endcodeblock %}  

Initialization of class objects/instances is handled by the ` __init__ ` method which is similar to the constructor in other languages.

{% codeblock %}
>>> class Square(object):
...    """Square is an object"""
...    totalSquares = 0
...    def __init__(self,length):
...         self.len = length
...         if Square.totalSquares == 0:
...            Square.totalSquares = 1
...         else:
...            Square.totalSquares += 1
...
>>> a = Square(10)
>>> b = Square(20)
>>> a.len
10
>>> b.len
20
>>> Square.totalSquares
2
>>> a.totalSquares
2
>>> b.totalSquares
2 
{% endcodeblock %}

Note that __init__ method attributes includes "self" as the first paramater which is a reserved word. This is applicable to all the method definitions in a class. But when a class object is created the first parameter starts with the second parameter in the __init__ method definition. The first parameter "self" is taken care by the Python interpreter. 

The variable "totalSquares" which is defined outside all the methods in the class definition is a class variable and shared by all the instances. The variable "self.len" is an instance level variable and is accessible to all the methods in the class. Other variables defined in a method are local variables to the method and not accessible outside of the method. The following is an expanded example

{% codeblock %}
class Square(object):
    """Square is an object"""
    totalSquares = 0                     # Class variable
    def __init__(self,length):
         self.len = length               # Instance variable
         if Square.totalSquares == 0:
            Square.totalSquares = 1
         else:
            Square.totalSquares += 1

    def area(self):
        a = self.len * self.len          # Local variable
        return a

    def printSquare(self):
        print "Length of square is %d" % self.len
        print "Area of square is %d" % self.area()
{% endcodeblock %} 

A class can inherit from multiple other classes. For e.g.

{% codeblock %}
class Cube(Square, Point):
    """Inherits square and point classes"""
...
{% endcodeblock %}

Similar to languages like Java, Python has a garbage collector (GC) to delete objects which are not referenced anymore. This prevents memory leaks by not releasing the ojects explicitly by programs as languages like C requires.
 
Parent class methods can be **overridden** by an inherited class by defining the same method header in the child class as in the parent class. As seen before the **__init__** defined in all the class definitions earlier is overriding the method with same header in "object" class which is the inbuild base class. Other base class methods which can be overridden are

`` def __del__(self): `` destructor method which is called at the time of GC of the class object
`` def __repr__(self): `` returns an evaluatable string represention of the class object
`` def __str__(self): ``  returns a string representation of the object like "toString" method in Java

If a method is overridden in a class but need to invoke the same method in the parent class, **super** can be used as in the following example. Calling super is bit different to that of other languaes.

{% codeblock %}
class Song(object):
     def __init__(self,name,lyrics):
         self.name = name
         self.lyrics = lyrics
     def write(self):
         print self.name, self.lyrics

class KSong(Song):
    def __init__(self,name,lyrics,time):
         super(KSong, self).__init__(name,lyrics)
         self.time = time
{% endcodeblock %}

Type of a class instance can be checked using the ` isinstance ` function. For e.g. assuming that the previous code is in a file song.py 

{% codeblock %}
>>> import song
True
>>> ks = song.KSong("Kel","Kan",10)
>>> isinstance(ks, song.KSong)
True
{% endcodeblock %}

Operator **overloading** is accomplished by overriding the base class methods provided to overload operators.

`` def __cmp__(self,other): `` comparing two class objects and returns -ve int if self < other, 0 if self == other and +ve int if self > other
`` def __add__(self,other): `` to overload addition (+)
`` def __sub__(self,other): `` to overload subtraction (-)

...
More details are available in [Python docs](https://docs.python.org/2/reference/datamodel.html)

This is just the tip of the iceberg. You can explore more about Python [here](https://docs.python.org/3/).
