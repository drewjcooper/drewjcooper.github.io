---
title: Exercise 3 - Collections and Sorting
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
---
The third exercise in [15 Exercises for Learning a new Programming
Language](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml)
is:

> Accepting series of numbers, strings from keyboard and sorting them
> ascending, descending order.

The naive solution would be to build an array of entered strings/numbers
and then write a method that will sort that array - a simple bubble sort
would do.  Of course, .NET offers a better way...
<!--more-->

The `System.Collections` and `System.Collections.Generic` namespaces
contain a number of classes that implement such things as dynamically
sized arrays, lists, ordered lists, hash tables, etc.  These classes
provide methods for sorting the contents of the collections.  For this
solution I decided to use a `List<T>` to store the input data and do the
sorting.  Here's the code:

{% highlight csharp linenos %}
using System;
using System.Collections.Generic;
using System.Linq;

class MainClass
{
    static void Main()
    {
        string inputLine;

        List<string> strings = new List<string>();
        Console.WriteLine("Enter a series of strings, one per line.  Finish with a blank line");
        while ((inputLine = Console.ReadLine()) != "")
        {
            strings.Add(inputLine);
        }
        strings.Sort();
        Console.WriteLine("Ascending list of strings:");
        foreach (string item in strings)
        {
            Console.WriteLine("   " + item);
        }
        strings.Reverse();
        Console.WriteLine("Descending list of strings:");
        foreach (string item in strings)
        {
            Console.WriteLine("   " + item);
        }

        Console.WriteLine();

        List<double> numbers = new List<double>();
        double inputNumber;
        Console.WriteLine("Enter a series of numbers, one per line.  Finish with a blank line");
        while ((inputLine = Console.ReadLine()) != "")
        {
            if (double.TryParse(inputLine, out inputNumber))
            {
                numbers.Add(inputNumber);
            }
        }
        numbers.Sort();
        Console.WriteLine("Ascending list of numbers:");
        foreach (double item in numbers)
        {
            Console.WriteLine("   " + item.ToString());
        }
        numbers.Reverse();
        Console.WriteLine("Descending list of numbers:");
        foreach (double item in numbers)
        {
            Console.WriteLine("   " + item.ToString());
        }
    }
}
{% endhighlight %}

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
ago]({% post_url 2010-06-23-exercise-2-fibonacci-and-other-things %}),
except that it doesn't require the variable used in the method call to
be initialised at the time of the call.

So, that will for this post, I think.  Till next time....
