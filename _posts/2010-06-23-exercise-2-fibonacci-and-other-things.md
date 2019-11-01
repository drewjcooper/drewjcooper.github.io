---
title: Exercise 2 – Fibonacci, and other things
date: '2010-06-23T22:22:00.001+10:00'
tags:
- C# Exercise
modified_time: '2010-06-23T22:47:11.769+10:00'
---
Prashant's [second
exercise](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml)
is:

> Fibonacci series, swapping two variables, finding maximum/minimum
> among a list of numbers.

So in this post we get to play with a bit of recursion, and I've thrown
in some parametric types for good measure.
<!--more-->

The code I came up with for this exercise is as follows:

{% highlight csharp linenos %}
using System;

class MainClass
{
    static ulong Fibonacci1(uint N)
    {
        switch (N)
        {
            case 0:
                return 0;
            case 1:
                return 1;
            default:
                ulong FN = 1, FNMinusTwo, FNMinusOne = 0;
                while (N > 1)
                {
                    FNMinusTwo = FNMinusOne;
                    FNMinusOne = FN;
                    FN = FNMinusTwo + FNMinusOne;
                    N--;
                }
                return FN;
        }
    }

    static ulong Fibonacci2(uint N, ulong NMinusTwo = 0, ulong NMinusOne = 1)
    {
        switch (N)
        {
            case 0:
                return NMinusTwo;
            case 1:
                return NMinusOne;
            default:
                return Fibonacci2(N - 1, NMinusOne, NMinusOne + NMinusTwo);
        }
    }

    static void Swap<T>(ref T var1, ref T var2) {
        T tempvar = var1;
        var1 = var2;
        var2 = tempvar;
    }

    static int Max(int[] Numbers) {
        int maxint = Numbers[0];
        foreach (int tempint in Numbers) {
            if (tempint > maxint) maxint = tempint;
        }
        return maxint;
    }

    static int Min(int[] Numbers)
    {
        int minint = Numbers[0];
        foreach (int tempint in Numbers)
        {
            if (tempint < minint) minint = tempint;
        }
        return minint;
    }

    static void Main()
    {
        Console.WriteLine("Fibonacci series");
        for (uint i = 0; i <= 10; i++)
        {
            Console.WriteLine("F(" + i.ToString() + ")\t" + Fibonacci1(i).ToString() + "\t" + Fibonacci2(i).ToString());
        }

        Console.WriteLine ("\nSwapping int Variables");
        int A = 5;
        int B = 10;
        Console.WriteLine("Before:\tA=" + A.ToString() + "\tB=" + B.ToString());
        Swap(ref A, ref B);
        Console.WriteLine("Before:\tA=" + A.ToString() + "\tB=" + B.ToString());

        Console.WriteLine("\nSwapping char Variables");
        char C = 'C';
        char D = 'D';
        Console.WriteLine("Before:\tC=" + C.ToString() + "\tD=" + D.ToString());
        Swap(ref C, ref D);
        Console.WriteLine("Before:\tC=" + C.ToString() + "\tD=" + D.ToString());

        int[] IntArray =  { 15, 4, 76, 53, 25, 63 };
        Console.WriteLine("\nArray = { 15, 4, 76, 53, 25, 63 }");
        Console.WriteLine("Maximum = " + Max(IntArray).ToString());
        Console.WriteLine("Minimum = " + Min(IntArray).ToString());
    }
}
{% endhighlight %}

For the Fibonacci numbers I decided to write two methods, each of which
take an integer N, and return the Nth number in the Fibonacci series. 
The first method does this using a loop.  The second is recursive and is
a lot more brief.  The methods show a few interesting aspects of C#.

Line 14 declares 3 variables of type **ulong** (UInt64) which are then
used to build the Fibonacci series.  In C# all variables must be
initialised before they are used in an expression.  Failure to do so is
a compile-time error.  Line 14 initialises `FN` and `FNMinusOne` as they
are declared, but leaves `FNMinusTwo` uninitialised.  This is okay
because `FNMinusTwo` is given a value in Line 17 before its first use in
line 19.

The other thing to notice about this method is that a parameter of a
***value type*** gets a copy of the value that is passed to it.  Thus,
although the method modifies the value of the parameter `N`, it doesn't
affect the value of the loop variable used in the `Main` function to
print the series.

The second Fibonacci function at line 26 shows the use of optional
parameters.  In C#, method parameters may be assigned a value in the
method declaration.  If the parameter is not passed a value when the
method is called it gets the value set in the declaration.  In this way
the initial call to `Fibonacci2()` only has to specify `N`, and the
initial value of the recursion parameters is specified in the method
declaration.  Of course, you could also specify values for these
parameters in the call to the method, in which case the method would
produce a different series of numbers.

In order to solve the problem of swapping two variables I decided to
create a generic method using type parameters.  This method can be used
to swap any two variables regardless of type, as long as they're the
same type.  The method declaration on line 39 creates a type parameter
`T` which is bound to a specific type when the method is called
elsewhere the code.  Notice that the calls to the method in lines 75 to
82 are identical, except for the variables used.  The types of the
variables bind to the type parameter at compile time.  Within the
method, variables of the parametric type are only be able to be used as
if they were of type `object`.

The other thing to note in the Swap method is the use of the `ref`
keyword in declaring the parameters, and passing the variables to them.
Without the `ref` keyword, variables of value types would be passed in
as a copy, so that, even though the values of the variable are swapped
inside the method, the values on the outside would stay the same.  The
ref keyword specifies that a reference to the variable is passed
instead, so that a change to the parameter inside the method, actually
changes the storage location of the variable outside the method.

The last part of the exercise is to find the maximum and minimum of a
list of numbers.  There's nothing really special in my implementation,
except for the use of the `foreach` statement.  That's a construct I've
enjoyed using in VB, and have always missed when I've been writing in
C.  It's basically a shortcut to enumerating the items in a collection,
without having to worry about dealing with indexes or enumerators.

The only other thing of note in this code is that the methods are all
declared static.  That's required to enable me to call the methods
without instantiating the `MainClass` class as an object.

So that's it for this post.  Next time I'll look at exercise 3.
