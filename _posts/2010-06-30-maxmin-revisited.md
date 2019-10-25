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
post](http://learningcsharpnet.blogspot.com/2010/06/exercise-2-fibonacci-and-other-things.html)
(thanks Matt) told me about an alternative to my Max/Min
implementation.  The .NET framework includes a class library called LINQ
(Language INtegrated Query).  In looking through the MSDN library I had
seen this mentioned, but figured it had something to do with database
queries.  I was partly right, but it turns out that LINQ is far more,
and allows an alternate solution to [Prashant\'s second
exercise](http://www.jobsnake.com/seek/articles/index.cgi?openarticle&8533).

[]{#more}

The `System.Linq` namespace includes classes with methods to query any
object that implements the IEnumerable interface.  This means that an
Array can be queried, just as a database table can be.  I'll discuss
this a bit more in a minute, but right now here is the solution from
last post rewritten to use LINQ.

::: {.csharpcode}
       1:  using System;

       2:  using System.Linq;

       3:   

       4:  class MainClass

       5:  {

       6:      static ulong Fibonacci1(uint n)

       7:      {

       8:          switch (n)

       9:          {

      10:              case 0:

      11:                  return 0;

      12:              case 1:

      13:                  return 1;

      14:              default:

      15:                  ulong fN = 1, fNMinusTwo, fNMinusOne = 0;

      16:                  while (n > 1)

      17:                  {

      18:                      fNMinusTwo = fNMinusOne;

      19:                      fNMinusOne = fN;

      20:                      fN = fNMinusTwo + fNMinusOne;

      21:                      n--;

      22:                  }

      23:                  return fN;

      24:          }

      25:      }

      26:      

      27:      static ulong Fibonacci2(uint n, ulong nMinusOne = 1, ulong nMinusTwo = 0)

      28:      {

      29:          switch (n)

      30:          {

      31:              case 0:

      32:                  return nMinusTwo;

      33:              case 1:

      34:                  return nMinusOne;

      35:              default:

      36:                  return Fibonacci2(n - 1, nMinusOne + nMinusTwo, nMinusOne);

      37:          }

      38:      }

      39:   

      40:      static void Swap<T>(ref T var1, ref T var2) {

      41:          T tempVar = var1;

      42:          var1 = var2;

      43:          var2 = tempVar;

      44:      }

      45:      

      46:      static void Main()

      47:      {

      48:          Console.WriteLine("Fibonacci series");

      49:          for (uint index = 0; index <= 10; index++)

      50:          {

      51:              Console.WriteLine("F(" + index.ToString() + ")\t" + Fibonacci1(index).ToString() + "\t" + Fibonacci2(index).ToString());

      52:          }

      53:   

      54:          Console.WriteLine ("\nSwapping int Variables");

      55:          int a = 5;

      56:          int b = 10;

      57:          Console.WriteLine("Before:\tA=" + a.ToString() + "\tB=" + b.ToString());

      58:          Swap(ref a, ref b);

      59:          Console.WriteLine("Before:\tA=" + a.ToString() + "\tB=" + b.ToString());

      60:          

      61:          Console.WriteLine("\nSwapping char Variables");

      62:          char c = 'C';

      63:          char d = 'D';

      64:          Console.WriteLine("Before:\tC=" + c.ToString() + "\tD=" + d.ToString());

      65:          Swap(ref c, ref d);

      66:          Console.WriteLine("Before:\tC=" + c.ToString() + "\tD=" + d.ToString());

      67:   

      68:          int[] integerArray =  { 15, 4, 76, 53, 25, 63 };

      69:          Console.Write("\nArray = {");

      70:          for (uint index = 0; index < integerArray.Count(); Console.Write(" " + integerArray[index++].ToString() + ","));

      71:          Console.WriteLine("\b }");

      72:          Console.WriteLine("Maximum = " + integerArray.Max().ToString());

      73:          Console.WriteLine("Minimum = " + integerArray.Min().ToString());

      74:      }

      75:  }
:::

Notice that the Min and Max methods are gone.  Instead, the array is
queried directly, using what appears to be a method of the Array.  In
actual fact `Max()` and `Min()` are **extension methods** provided by
the System.Linq.Enumerable class.   The `using System.Linq;` statement
in line 2 brings these extension methods, and many others, into scope so
they can be used directly from the array object.

Matt also pointed me to Microsoft's [Capitalization
Conventions](http://msdn.microsoft.com/en-us/library/ms229043.aspx) for
use in developing class libraries.  I've renamed some of the variables
and parameters in this solution to adhere to these conventions.

#### More on Extension Methods

To quote the [C# Programming
Guide](http://msdn.microsoft.com/en-us/library/67ef8sbd.aspx) in MSDN:

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

```csharppublic static int Max(
    this IEnumerable<int> source
)
```

This means that this method is usable on any type that implements
IEnumerable\<int\>, and there are versions of this extension method for
each of the other numeric types, as well as a generic one in which you
can specify a comparison method.

I'll leave that here for now.  Next time I'll address Prashant's third
exercise.
