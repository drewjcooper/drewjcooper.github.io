---
title: 'Exercise 5 – Reynolds again: overflow without exception'
date: '2010-07-13T22:53:00.001+10:00'
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
modified_time: '2010-07-13T22:53:46.940+10:00'
---
The 5th of Prashant's [15 Exercises for Learning a new Programming
Language](http://www.jobsnake.com/seek/articles/index.cgi?openarticle&8533)
builds directly onto the
[4th](http://learningcsharpnet.blogspot.com/2010/07/exercise-4-reynolds-number-basic-class.html):

> Modify the above program such that it will ask for \'Do you want to
> calculate again (y/n),\
> if you say \'y\', it\'ll again ask the parameters. If \'n\', it\'ll
> exit. (Do while loop)
>
> While running the program give value mu = 0. See what happens. Does it
> give \'DIVIDE BY ZERO\' error?\
> Does it give \'Segmentation fault..core dump?\'. How to handle this
> situation. Is there something built\
> in the language itself? (Exception Handling)

The first bit is easy, the second bit may be surprising.  First thing
I'll do is show you what happens when I run the program from last time
with mu = 0.

[]{#more}

``` {.console}
Calculate Reynold's Number and flow characteristic
Enter the Density (?):  15
Enter the Diameter (D):  15
Enter the Velocity (v):  15
Enter the Viscosity (µ):  0
Reynold's Number = Infinity (Turbulent flow)
```

Well, how about that?  No "divide by zero" exception; no crash and burn;
just an infinite result.  In fact, floating point operations in C#
never throw an exception -- they return either a valid number, or one of
two special values, **Infinity** and **NaN** (Not a Number).  It's even
possible to enter these values and have them parse correctly as floating
point values as shown in this next program run:

``` {.console}
Calculate Reynold's Number and flow characteristic
Enter the Density (?):  Infinity
Enter the Diameter (D):  4
Enter the Velocity (v):  5
Enter the Viscosity (µ):  Infinity
Reynold's Number = NaN (Turbulent flow)
```

As shown above, a divide by zero results in the value **Infinity**, and
a divide by Infinity results in NaN.  In general, any overflow will
result in the value **Infinity** as shown below:

``` {.console}
Calculate Reynold's Number and flow characteristic
Enter the Density (?):  1E200
Enter the Diameter (D):  1E200
Enter the Velocity (v):  1E200
Enter the Viscosity (µ):  1
Reynold's Number = Infinity (Turbulent flow)
```

The true result should be 1E+600, but that is too large to be
represented by a double, so we get **Infinity** instead.

Even though we don't have an exception or crash, it's still not nice
that we get an infinite result but still have a valid flow type
returned.  Also, there are a number of quantities in the Reynolds
formula for which only a positive non-zero number make sense, one of
which is the problematic Viscosity, so it would be good to filter the
input of these values to something vaguely reasonable.

So with those thoughts, and the first part of the exercise in mind I
present the code for this post:

::: {.csharpcode}
       1:  using System;

       2:   

       3:  class MainClass

       4:  {

       5:      static double ReadNonNegativeDouble(string inputPrompt = "Enter a number", bool zeroAllowed = false)

       6:      {

       7:          double inputValue;

       8:   

       9:          do

      10:          {

      11:              Console.Write(inputPrompt + ":  ");

      12:              if (double.TryParse(Console.ReadLine(), out inputValue))

      13:              {

      14:                  if (inputValue > 0 || inputValue == 0 && zeroAllowed) break;

      15:              }

      16:          }

      17:          while (true);

      18:          return inputValue;

      19:      }

      20:   

      21:      static void Main()

      22:      {

      23:          Reynolds reynolds = new Reynolds();

      24:          Console.WriteLine("\nCalculate Reynold's Number and flow characteristic\n");

      25:          do

      26:          {

      27:              reynolds.density = ReadNonNegativeDouble("Enter the Density (\x03c1)");

      28:              reynolds.diameter = ReadNonNegativeDouble("Enter the Diameter (D)");

      29:              reynolds.velocity = ReadNonNegativeDouble("Enter the Velocity (v)", true);

      30:              reynolds.viscosity = ReadNonNegativeDouble("Enter the Viscosity (\x03bc)");

      31:              if (double.IsInfinity(reynolds.Number)) 

      32:              {

      33:                  Console.WriteLine("Calculation overflow!");

      34:              }

      35:              else if (double.IsNaN(reynolds.Number))

      36:              {

      37:                  Console.WriteLine("Invalid result!");

      38:              }

      39:              else

      40:              {

      41:                  Console.WriteLine("Reynold's Number = " + reynolds.Number.ToString() + " (" + reynolds.FlowType + " flow)");

      42:              }

      43:              Console.Write("\nDo you want to calculate again? (y/n) ");

      44:              while (true) {

      45:                  char keyPress = Console.ReadKey(true).KeyChar;

      46:                  if (keyPress == 'y') break;

      47:                  if (keyPress == 'n') return;

      48:              }

      49:              Console.Write("\n\n");

      50:          } while (true);

      51:      }

      52:  }

      53:   

      54:  class Reynolds

      55:  {

      56:      public double density;

      57:      public double diameter;

      58:      public double velocity;

      59:      public double viscosity;

      60:   

      61:      public double Number

      62:      {

      63:          get

      64:          {

      65:              return density * diameter * velocity / viscosity;

      66:          }

      67:      }

      68:   

      69:      public string FlowType

      70:      {

      71:          get

      72:          {

      73:              if (double.IsInfinity(Number) || double.IsNaN(Number)) 

      74:              {

      75:                  return "Invalid result";

      76:              }

      77:              else if (Number < 2100)

      78:              {

      79:                  return "Laminar";

      80:              }

      81:              else if (Number < 4000)

      82:              {

      83:                  return "Transient";

      84:              }

      85:              else

      86:              {

      87:                  return "Turbulent";

      88:              }

      89:          }

      90:      }

      91:  }
:::

The only thing really worthy of note in this code is the use of two
**static methods** of the `double` class, `double.IsInfinity()` and
`double.IsNaN()`, which check a variable for these two special values. 
Here's the result of these changes.

``` {.console}
Calculate Reynold's Number and flow characteristic

Enter the Density (?):  15
Enter the Diameter (D):  15
Enter the Velocity (v):  15
Enter the Viscosity (µ):  3
Reynold's Number = 1125 (Laminar flow)

Do you want to calculate again? (y/n)

Enter the Density (?):  23
Enter the Diameter (D):  34
Enter the Velocity (v):  Infinity
Enter the Viscosity (µ):  Infinity
Invalid result!

Do you want to calculate again? (y/n)

Enter the Density (?):  32
Enter the Diameter (D):  23
Enter the Velocity (v):  43
Enter the Viscosity (µ):  0
Enter the Viscosity (µ):  4
Reynold's Number = 7912 (Turbulent flow)

Do you want to calculate again? (y/n)
```

Next post will be the first of a small series which cover Exercise 6 of
Prashant's exercises.  The solution I've developed has about 450 lines
of code so it will take a few posts to get through it all.  See you
then.
