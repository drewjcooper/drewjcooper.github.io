---
title: 'Exercise 5 – Reynolds again: overflow without exception'
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
---
The 5th of Prashant's [15 Exercises for Learning a new Programming
Language](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml)
builds directly onto the
[4th](http://learningcsharpnet.blogspot.com/2010/07/exercise-4-reynolds-number-basic-class.html):

> Modify the above program such that it will ask for 'Do you want to calculate
> again (y/n), if you say 'y', it'll again ask the parameters. If 'n', it'll
> exit. (Do while loop)
>
> While running the program give value mu = 0. See what happens. Does it give
> 'DIVIDE BY ZERO' error? Does it give 'Segmentation fault..core dump?'. How to
> handle this situation. Is there something built in the language itself?
> (Exception Handling)

The first bit is easy, the second bit may be surprising.  First thing I'll do is
show you what happens when I run the program from last time with mu = 0.
<!--more-->

```text
Calculate Reynold's Number and flow characteristic
Enter the Density (?):  15
Enter the Diameter (D):  15
Enter the Velocity (v):  15
Enter the Viscosity (µ):  0
Reynold's Number = Infinity (Turbulent flow)
```

Well, how about that?  No "divide by zero" exception; no crash and burn; just an
infinite result. In fact, floating point operations in C# never throw an
exception - they return either a valid number, or one of two special values,
**Infinity** and **NaN** (Not a Number).  It's even possible to enter these
values and have them parse correctly as floating point values as shown in this
next program run:

```text
Calculate Reynold's Number and flow characteristic
Enter the Density (?):  Infinity
Enter the Diameter (D):  4
Enter the Velocity (v):  5
Enter the Viscosity (µ):  Infinity
Reynold's Number = NaN (Turbulent flow)
```

As shown above, a divide by zero results in the value **Infinity**, and a divide
by Infinity results in NaN.  In general, any overflow will result in the value
**Infinity** as shown below:

```text
Calculate Reynold's Number and flow characteristic
Enter the Density (?):  1E200
Enter the Diameter (D):  1E200
Enter the Velocity (v):  1E200
Enter the Viscosity (µ):  1
Reynold's Number = Infinity (Turbulent flow)
```

The true result should be 1E+600, but that is too large to be represented by a
double, so we get **Infinity** instead.

Even though we don't have an exception or crash, it's still not nice that we get
an infinite result but still have a valid flow type returned.  Also, there are a
number of quantities in the Reynolds formula for which only a positive non-zero
number make sense, one of which is the problematic Viscosity, so it would be
good to filter the input of these values to something vaguely reasonable.

So with those thoughts, and the first part of the exercise in mind I present the
code for this post:

```csharp
using System;

class MainClass
{
    static double ReadNonNegativeDouble(string inputPrompt = "Enter a number", bool zeroAllowed = false)
    {
        double inputValue;

        do
        {
            Console.Write(inputPrompt + ":  ");
            if (double.TryParse(Console.ReadLine(), out inputValue))
            {
                if (inputValue > 0 || inputValue == 0 && zeroAllowed) break;
            }
        }
        while (true);
        return inputValue;
    }

    static void Main()
    {
        Reynolds reynolds = new Reynolds();
        Console.WriteLine("\nCalculate Reynold's Number and flow characteristic\n");
        do
        {
            reynolds.density = ReadNonNegativeDouble("Enter the Density (\x03c1)");
            reynolds.diameter = ReadNonNegativeDouble("Enter the Diameter (D)");
            reynolds.velocity = ReadNonNegativeDouble("Enter the Velocity (v)", true);
            reynolds.viscosity = ReadNonNegativeDouble("Enter the Viscosity (\x03bc)");
            if (double.IsInfinity(reynolds.Number)) 
            {
                Console.WriteLine("Calculation overflow!");
            }
            else if (double.IsNaN(reynolds.Number))
            {
                Console.WriteLine("Invalid result!");
            }
            else
            {
                Console.WriteLine("Reynold's Number = " + reynolds.Number.ToString() + " (" + reynolds.FlowType + " flow)");
            }
            Console.Write("\nDo you want to calculate again? (y/n) ");
            while (true) {
                char keyPress = Console.ReadKey(true).KeyChar;
                if (keyPress == 'y') break;
                if (keyPress == 'n') return;
            }
            Console.Write("\n\n");
        } while (true);
    }
}

class Reynolds
{
    public double density;
    public double diameter;
    public double velocity;
    public double viscosity;

    public double Number
    {
        get
        {
            return density * diameter * velocity / viscosity;
        }
    }

    public string FlowType
    {
        get
        {
            if (double.IsInfinity(Number) || double.IsNaN(Number)) 
            {
                return "Invalid result";
            }
            else if (Number < 2100)
            {
                return "Laminar";
            }
            else if (Number < 4000)
            {
                return "Transient";
            }
            else
            {
                return "Turbulent";
            }
        }
    }
}
```

The only thing really worthy of note in this code is the use of two **static
methods** of the `double` class, `double.IsInfinity()` and `double.IsNaN()`,
which check a variable for these two special values. Here's the result of these
changes.

```text
Calculate Reynold's Number and flow characteristic

Enter the Density (ρ):  15
Enter the Diameter (D):  15
Enter the Velocity (v):  15
Enter the Viscosity (µ):  3
Reynold's Number = 1125 (Laminar flow)

Do you want to calculate again? (y/n)

Enter the Density (ρ):  23
Enter the Diameter (D):  34
Enter the Velocity (v):  Infinity
Enter the Viscosity (µ):  Infinity
Invalid result!

Do you want to calculate again? (y/n)

Enter the Density (ρ):  32
Enter the Diameter (D):  23
Enter the Velocity (v):  43
Enter the Viscosity (µ):  0
Enter the Viscosity (µ):  4
Reynold's Number = 7912 (Turbulent flow)

Do you want to calculate again? (y/n)
```

Next post will be the first of a small series which cover Exercise 6 of
Prashant's exercises.  The solution I've developed has about 450 lines of code
so it will take a few posts to get through it all.  See you then.
