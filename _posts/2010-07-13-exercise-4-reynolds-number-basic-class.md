---
title: 'Exercise 4 – Reynold’s Number: a basic class'
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
---
The fourth of [Prashant's
exercises](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml)
is:

> Reynolds number is calculated using formula (D\*v\*rho)/mu Where D = Diameter,
> V= velocity, rho = density, mu = viscosity. Write a program that will accept
> all values in appropriate units (Don't worry about unit conversion). If number
> is < 2100, display Laminar flow, If it's between 2100 and 4000 display
> 'Transient flow' and if more than '4000', display 'Turbulent Flow' (If, else,
> then...)

In working this solution I decided to declare a class to help in the
calculation.  The solution also shows some details of reading input from the
console.
<!--more-->

For this post I've broken the code into two boxes.  The first, as shown by the
line numbers, is actually the second part of the program, but I want to explain
it first.

```csharp
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
            if (Number < 2100)
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

This is the declaration of the Reynolds **class**.  The class has 4 **fields**
which hold values for the four variables in the equation, and two
**properties**, `Number` and `FlowType`, which return values based on the
calculation of the formula.  As you'll see below, properties of a class are
accessed that same way as fields, but as seen above they can contain code to
calculate, or otherwise determine the value to be returned.  The two properties
here are **read-only properties** because they only have a `get` clause.  It is
also possible to create **write-only properties** (only a `set` clause) and
**read-write properties** (both `get` and `set`).  Note that all the **fields**
and **properties** are declared `public` so that they're accessible from outside
the class.  Common code conventions generally dictate that fields should be
declared `private`, and that applicable `public`, `protected`, or `internal`
properties should be created to allow the necessary access.

The second code block shows the main part of the program (no pun intended).

```csharp
using System;

class MainClass
{
    static double ReadDouble (string inputPrompt = "Enter a number")
    {
        double inputValue;

        do
        {
            Console.Write(inputPrompt + ":  ");
        }
        while (!double.TryParse(Console.ReadLine(), out inputValue));
        return inputValue;
    }

    static void Main()
    {
        Reynolds reynolds = new Reynolds();
        Console.WriteLine("Calculate Reynold's Number and flow characteristic");
        reynolds.density = ReadDouble("Enter the Density (\x03c1)");
        reynolds.diameter = ReadDouble("Enter the Diameter (D)");
        reynolds.velocity = ReadDouble("Enter the Velocity (v)");
        reynolds.viscosity = ReadDouble("Enter the Viscosity (\x03bc)");
        Console.WriteLine("Reynold's Number = " + reynolds.Number.ToString() + " (" + reynolds.FlowType + " flow)");
    }
}
```

To keep the code simple I created a method (`ReadDouble`) to prompt the user for
a value from the console, and continue to prompt until a valid `double` value is
entered.  We've seen **optional parameters** before. Here I've declared
`inputPrompt` with a default prompt that will be used if none is specified in
the method call.  This method also shows use of C#'s `do {} while ()` loop
construct, which should be immediately familiar to any C/C++ programmer.  C#
also has a `while () {}` construct, with the loop condition at the front-end.

There's nothing terribly special about the `Main` method.  First thing it does
is to create an instance of the `Reynolds` **class**, and assign it to a
**variable** called `reynolds`.  Note that the class and variable names differ
only by case.  As C# is case-sensitive language this isn't a problem, but it is
frowned upon because it can lead to problems when interoperability with a
case-insensitive language, like VB.NET, is required.

The `ReadDouble` method is called a number of times to prompt for each of the
required values in turn, and the returned values are assigned to the appropriate
fields in the Reynolds class.  One thing to note here is the use of Unicode
escape sequences in a couple of the prompt strings. These are the codes for the
greek letters mu and rho.  All strings in C# are Unicode strings, so there's no
need to play with different types and functions as in C/C++.

In reading through the code above you probably thought, "hang about, what
happens if I enter zero for the Viscosity (the divisor)?"  That's a good
question because most languages would throw an exception in that case, and I
haven't done anything to trap that.  You're going to have to wait for the next
post for the answer, because that is actually the subject of Exercise 5.
