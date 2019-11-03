---
title: Max/Min revisited
date: '2010-06-30T22:50:00.001+10:00'
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
modified_time: '2010-06-30T22:50:14.399+10:00'
---
A kind commenter on [my last
post]({% post_url 2010-06-23-exercise-2-fibonacci-and-other-things %})
(thanks Matt) told me about an alternative to my Max/Min
implementation.  The .NET framework includes a class library called LINQ
(Language INtegrated Query).  In looking through the MSDN library I had
seen this mentioned, but figured it had something to do with database
queries.  I was partly right, but it turns out that LINQ is far more,
and allows an alternate solution to [Prashant's second
exercise](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml).
<!--more-->

The `System.Linq` namespace includes classes with methods to query any
object that implements the IEnumerable interface.  This means that an
Array can be queried, just as a database table can be.  I'll discuss
this a bit more in a minute, but right now here is the solution from
last post rewritten to use LINQ.

```csharp
using System;
using System.Linq;

class MainClass
{
    static ulong Fibonacci1(uint n)
    {
        switch (n)
        {
            case 0:
                return 0;
            case 1:
                return 1;
            default:
                ulong fN = 1, fNMinusTwo, fNMinusOne = 0;
                while (n > 1)
                {
                    fNMinusTwo = fNMinusOne;
                    fNMinusOne = fN;
                    fN = fNMinusTwo + fNMinusOne;
                    n--;
                }
                return fN;
        }
    }

    static ulong Fibonacci2(uint n, ulong nMinusOne = 1, ulong nMinusTwo = 0)
    {
        switch (n)
        {
            case 0:
                return nMinusTwo;
            case 1:
                return nMinusOne;
            default:
                return Fibonacci2(n - 1, nMinusOne + nMinusTwo, nMinusOne);
        }
    }

    static void Swap<T>(ref T var1, ref T var2) {
        T tempVar = var1;
        var1 = var2;
        var2 = tempVar;
    }

    static void Main()
    {
        Console.WriteLine("Fibonacci series");
        for (uint index = 0; index <= 10; index++)
        {
            Console.WriteLine("F(" + index.ToString() + ")\t" + Fibonacci1(index).ToString() + "\t" + Fibonacci2(index).ToString());
        }

        Console.WriteLine ("\nSwapping int Variables");
        int a = 5;
        int b = 10;
        Console.WriteLine("Before:\tA=" + a.ToString() + "\tB=" + b.ToString());
        Swap(ref a, ref b);
        Console.WriteLine("Before:\tA=" + a.ToString() + "\tB=" + b.ToString());

        Console.WriteLine("\nSwapping char Variables");
        char c = 'C';
        char d = 'D';
        Console.WriteLine("Before:\tC=" + c.ToString() + "\tD=" + d.ToString());
        Swap(ref c, ref d);
        Console.WriteLine("Before:\tC=" + c.ToString() + "\tD=" + d.ToString());

        int[] integerArray =  { 15, 4, 76, 53, 25, 63 };
        Console.Write("\nArray = {");
        for (uint index = 0; index < integerArray.Count(); Console.Write(" " + integerArray[index++].ToString() + ","));
        Console.WriteLine("\b }");
        Console.WriteLine("Maximum = " + integerArray.Max().ToString());
        Console.WriteLine("Minimum = " + integerArray.Min().ToString());
    }
}
```

Notice that the Min and Max methods are gone.  Instead, the array is
queried directly, using what appears to be a method of the Array.  In
actual fact `Max()` and `Min()` are **extension methods** provided by
the `System.Linq.Enumerable` class.   The `using System.Linq;` statement
in line 2 brings these extension methods, and many others, into scope so
they can be used directly from the array object.

Matt also pointed me to Microsoft's [Capitalization
Conventions](https://msdn.microsoft.com/en-us/library/ms229043.aspx) for
use in developing class libraries.  I've renamed some of the variables
and parameters in this solution to adhere to these conventions.

###More on Extension Methods

To quote the [C# Programming
Guide](https://msdn.microsoft.com/en-us/library/67ef8sbd.aspx) in MSDN:

> Extension methods enable you to \"add\" methods to existing types
> without creating a new derived type, recompiling, or otherwise
> modifying the original type. Extension methods are a special kind of
> static method, but they are called as if they were instance methods on
> the extended type. For client code written in C# and Visual Basic,
> there is no apparent difference between calling an extension method
> and the methods that are actually defined in a type.

An extension method is basically a static method, but the first
parameter identifies the type that the method extends, and is preceded
by the `this` keyword.  So the `Max()` method used above is defined as
follows:

```csharp
public static int Max(
    this IEnumerable<int> source
)
```

This means that this method is usable on any type that implements
`IEnumerable<int>`, and there are versions of this extension method for
each of the other numeric types, as well as a generic one in which you
can specify a comparison method.

I'll leave that here for now.  Next time I'll address Prashant's third
exercise.
