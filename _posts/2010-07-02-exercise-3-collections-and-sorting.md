---
title: Exercise 3 - Collections and Sorting
date: '2010-07-02T04:41:00.001+10:00'
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
modified_time: '2010-07-02T04:58:25.866+10:00'
---
The third exercise in [15 Exercises for Learning a new Programming
Language](http://www.jobsnake.com/seek/articles/index.cgi?openarticle&8533)
is:

> Accepting series of numbers, strings from keyboard and sorting them
> ascending, descending order.

The naive solution would be to build an array of entered strings/numbers
and then write a method that will sort that array - a simple bubble sort
would do.  Of course, .NET offers a better way...

[]{#more}

The `System.Collections` and `System.Collections.Generic` namespaces
contain a number of classes that implement such things as dynamically
sized arrays, lists, ordered lists, hash tables, etc.  These classes
provide moethods for sorting the contents of the collections.  For this
solution I decided to use a `List<T>` to store the input data and do the
sorting.  Here's the code:

::: {.csharpcode}
       1:  using System;

       2:  using System.Collections.Generic;

       3:  using System.Linq;

       4:   

       5:  class MainClass

       6:  {

       7:      static void Main()

       8:      {

       9:          string inputLine;

      10:   

      11:          List<string> strings = new List<string>();

      12:          Console.WriteLine("Enter a series of strings, one per line.  Finish with a blank line"); 

      13:          while ((inputLine = Console.ReadLine()) != "") {

      14:              strings.Add(inputLine);

      15:          }

      16:          strings.Sort();

      17:          Console.WriteLine("Ascending list of strings:");

      18:          foreach (string item in strings)

      19:          {

      20:              Console.WriteLine("   " + item);

      21:          }

      22:          strings.Reverse();

      23:          Console.WriteLine("Descending list of strings:");

      24:          foreach (string item in strings)

      25:          {

      26:              Console.WriteLine("   " + item);

      27:          }

      28:   

      29:          Console.WriteLine();

      30:   

      31:          List<double> numbers = new List<double>();

      32:          double inputNumber;

      33:          Console.WriteLine("Enter a series of numbers, one per line.  Finish with a blank line");

      34:          while ((inputLine = Console.ReadLine()) != "")

      35:          {

      36:              if (double.TryParse(inputLine, out inputNumber))

      37:              {

      38:                  numbers.Add(inputNumber);

      39:              }

      40:          }

      41:          numbers.Sort();

      42:          Console.WriteLine("Ascending list of numbers:");

      43:          foreach (double item in numbers)

      44:          {

      45:              Console.WriteLine("   " + item.ToString());

      46:          }

      47:          numbers.Reverse();

      48:          Console.WriteLine("Descending list of numbers:");

      49:          foreach (double item in numbers)

      50:          {

      51:              Console.WriteLine("   " + item.ToString());

      52:          }

      53:      }

      54:  }
:::

Notice that this code is in two halves that are almost identical.  Lines
11 through 27 takes string input and add it to a list, then sort the
list and output, then reverse the elements in the list and output
again.  Lines 31 through 52 do the same for numbers (in this case
`double`). 

Aside from the variable names and the type parameter in the `List<T>`
type, the only real difference in the code between the two sections is
line 36.  Each of the numeric types have a static method called
`TryParse` which tries to parse a value of that type from a given
string, and returns `true` or `false` to indicate success or failure. 
There is also a `Parse` method which returns the parsed value directly,
but throws an exception if it can't find a value in the string.

One thing to notice about `TryParse` is the `out` modifier on the second
parameter.  This modifier is like the `ref` modifier we saw in the swap
method [a couple of posts
ago](http://learningcsharpnet.blogspot.com/2010/06/exercise-2-fibonacci-and-other-things.html),
except that it doesn't require the variable used in the method call to
be initialised at the time of the call.

So, that will for this post, I think.  Till next time....
