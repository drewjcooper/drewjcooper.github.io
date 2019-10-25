---
title: Exercise 6 (part 5) – Putting our DLL through it’s paces
date: '2011-04-17T23:09:00.001+10:00'
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
modified_time: '2011-04-17T23:09:31.863+10:00'
---
Over the last four posts I've built up the engine of an RPN calculator
in the form of a DLL.  Now it's finaly time to put it use.  In this post
I'll present the code for a console-based user interface for our
calculator.

[]{#more}

Before we look at the code I'll show you you the calculator interface I
ended up with.

``` {style="background-color: black; width: 450px; color: white; margin-left: 2em; font-size: 10px"}
                                        <0>     0       (Zero)
                                        <1>     1       (One)
4:                                      <2>     2       (Two)
3:                                      <3>     3       (Three)
2:                                      <4>     4       (Four)
1:                                      <5>     5       (Five)
                                        <6>     6       (Six)
_                                       <7>     7       (Seven)
                                        <8>     8       (Eight)
                                        <9>     9       (Nine)
                                        <.>     .       (Decimal)
                                        <E>     E       (Exponent)
                                        <BkSpc> Back    (Back)
                                        <Enter> Enter   (Enter)
                                        <_>     +/-     (Negate)
                                        <\>     1/x     (Inverse)
                                        <p>     π       (PI)
                                        <e>     e       (e)
                                        <+>     +       (Add)
                                        <->     -       (Subtract)
                                        <*>     x       (Multiply)
                                        </>     ÷       (Divide)
                                        <%>     mod     (Modulus)
                                        <!>     !       (Factorial)
                                        <c>     cos     (Cosine)
                                        <s>     sin     (Sine)
                                        <t>     tan     (Tangent)
                                        <S>     x²      (Square)
                                        <C>     x3      (Cube)
                                        <R>     √       (SquareRoot)
```

The left side of the console is the calculator display.  The top line is
used for messages.  The four numbered lines are the top four values on
the stack.  The line with the cursor is the input line. 

On the right side of the display are the calculator "buttons".  In this
interface they don't actually do anything -- they're just a visual
display of the key options available to the user.  The left column is
the key-stroke that activates that operation, the middle column is the
display text for the button face (this is what would be on the button in
a GUI), and the right-hand column is the name of the operation.

And now to the code.  In Visual Studio begin a new Console Application
project, and add a reference to the DLL.

::: {.csharpcode}
       1:  using System;

       2:  using System.Collections.Generic;

       3:   

       4:  using LearningCSharp;

       5:   

       6:  class ConsoleRPNCalculator

       7:  {

       8:      static RPNCalculatorEngine rpnCalcEngine;

       9:      static Dictionary<char, int> buttons;

      10:   
:::

We start with a class for the Console-based calculator UI.  The class
has static members for the calculator engine, and a dictionary of the
"buttons" for the interface.  This dictionary maps a key-stroke
character to the index of the operation in the calculator engine. 
`Dictionary<char, int>` is one of the many pre-defined generics-based
collections in the `System.Collections.Generic` namespace.

::: {.csharpcode}
      11:      static private void DrawCalculator()

      12:      {

      13:          for (int stackIndex = 3; stackIndex >= 0; stackIndex--)

      14:          {

      15:              Console.SetCursorPosition(0, 5 - stackIndex);

      16:              Console.Write("                         ");

      17:              Console.CursorLeft = 0;

      18:              Console.WriteLine((stackIndex + 1) + ": " + rpnCalcEngine[stackIndex]);

      19:          }

      20:          Console.SetCursorPosition(0, 7);

      21:          Console.Write("                         ");

      22:          Console.SetCursorPosition(0, 7);

      23:          Console.Write(rpnCalcEngine.InputString);

      24:      }

      25:   
:::

The `DrawCalculator` method draws the calculator interface on the
left-side of the UI, using the indexer and `InputString` properties of
the calculator engine.  This method shows some of the cursor positioning
capabilities of the `System.Console` class.  This class also provides
full control over text and background colours and the size of the
console.  It's easy to create a simple text-based interface that's not
just a scrolling list of input/output.

::: {.csharpcode}
      26:      static void DisplayButtons()

      27:      {

      28:          Console.Clear();

      29:          for (int buttonIndex = 0; buttonIndex < buttons.Count; buttonIndex++)

      30:          {

      31:              RPNCalculatorEngine.Operation operation = rpnCalcEngine.GetOperation(buttonIndex);

      32:              Console.SetCursorPosition(40,buttonIndex);

      33:              Console.CursorTop = buttonIndex;

      34:              string keyName = operation.ShortCut == '\b' ? "BkSpc" : operation.ShortCut == '\r' ? "Enter" : operation.ShortCut.ToString();

      35:              Console.Write("<" + keyName + ">\t" + operation.KeyText + "\t(" + operation.Name + ")");

      36:          }

      37:      }

      38:   
:::

The `DisplayButtons` method draws the right-hand side of the interface
listing the "buttons" and associated operations, by looping through the
list of operations provided by the engine.

::: {.csharpcode}
      39:      static void WriteErrorLine(string errorString)

      40:      {

      41:          Console.SetCursorPosition(0, 0);

      42:          Console.Write(errorString);

      43:      }

      44:   
:::

The WriteErrorLine method just writes the given message in the error
line of the interface.

::: {.csharpcode}
      45:      public static void Main()

      46:      {

      47:          rpnCalcEngine = new RPNCalculatorEngine();

      48:          buttons = new Dictionary<char, int>();

      49:          RPNCalculatorEngine.Operation operation;

      50:   

      51:          int i = 0;

      52:          bool errorDisplayed = false;

      53:          while ((operation = rpnCalcEngine.GetOperation(i)) != null)

      54:          {

      55:              buttons.Add(operation.ShortCut, i++);

      56:          }

      57:          DisplayButtons();

      58:          do

      59:          {

      60:              DrawCalculator();

      61:              int button;

      62:              char keyChar = Console.ReadKey(true).KeyChar;

      63:              if (keyChar == '\x1b') break;    // exit the loop if Escape key pressed

      64:              if (buttons.TryGetValue(keyChar, out button))

      65:              {

      66:                  try

      67:                  {

      68:                      if (errorDisplayed)

      69:                      {

      70:                          Console.SetCursorPosition(0, 0);

      71:                          WriteErrorLine("                                   ");

      72:                          WriteErrorLine("                                   ");

      73:                          DisplayButtons();

      74:                          errorDisplayed = false;

      75:                      }

      76:                      rpnCalcEngine.PressKey(button);

      77:                  }

      78:                  catch (FormatException)

      79:                  {

      80:                      WriteErrorLine("Invalid number format");

      81:                      errorDisplayed = true;

      82:                  }

      83:                  catch (Exception e)

      84:                  {

      85:                      WriteErrorLine(e.Message);

      86:                      errorDisplayed = true;

      87:                  }

      88:              }

      89:          } while (true);

      90:      }

      91:  }
:::

The good old `Main` method.  Here we set up the calculator interface and
just run through the main application loop -- reading keyboard input,
updating the calculator display, and displaying the error messages from
any exceptions thrown by the engine.

So that's it.  Next time I'll have a look at some of the things I've
been learning in ASP.Net MVC.
