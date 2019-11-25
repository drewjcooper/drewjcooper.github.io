---
title: Exercise 6 (part 5) – Putting our DLL through it’s paces
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
---
Over the last four posts I've built up the engine of an RPN calculator in the
form of a DLL.  Now it's finally time to put it use.  In this post I'll present
the code for a console-based user interface for our calculator.
<!--more-->

Before we look at the code I'll show you you the calculator interface I ended up
with.

```text
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

The left side of the console is the calculator display.  The top line is used
for messages. The four numbered lines are the top four values on the stack.
The line with the cursor is the input line.

On the right side of the display are the calculator "buttons".  In this
interface they don't actually do anything - they're just a visual display of the
key options available to the user.  The left column is the key-stroke that
activates that operation, the middle column is the display text for the button
face (this is what would be on the button in a GUI), and the right-hand column
is the name of the operation.

And now to the code.  In Visual Studio begin a new Console Application project,
and add a reference to the DLL.

```csharp
using System;
using System.Collections.Generic;

using LearningCSharp;

class ConsoleRPNCalculator
{
    static RPNCalculatorEngine rpnCalcEngine;
    static Dictionary<char, int> buttons;
```

We start with a class for the Console-based calculator UI.  The class has static
members for the calculator engine, and a dictionary of the "buttons" for the
interface.  This dictionary maps a key-stroke character to the index of the
operation in the calculator engine. `Dictionary<char, int>` is one of the many
pre-defined generics-based collections in the `System.Collections.Generic`
namespace.

```csharp
    static private void DrawCalculator()
    {
        for (int stackIndex = 3; stackIndex >= 0; stackIndex--)
        {
            Console.SetCursorPosition(0, 5 - stackIndex);
            Console.Write("                         ");
            Console.CursorLeft = 0;
            Console.WriteLine((stackIndex + 1) + ": " + rpnCalcEngine[stackIndex]);
        }
        Console.SetCursorPosition(0, 7);
        Console.Write("                         ");
        Console.SetCursorPosition(0, 7);
        Console.Write(rpnCalcEngine.InputString);
    }
```

The `DrawCalculator` method draws the calculator interface on the
left-side of the UI, using the indexer and `InputString` properties of
the calculator engine.  This method shows some of the cursor positioning
capabilities of the `System.Console` class.  This class also provides
full control over text and background colours and the size of the
console.  It's easy to create a simple text-based interface that's not
just a scrolling list of input/output.

```csharp
    static void DisplayButtons()
    {
        Console.Clear();
        for (int buttonIndex = 0; buttonIndex < buttons.Count; buttonIndex++)
        {
            RPNCalculatorEngine.Operation operation = rpnCalcEngine.GetOperation(buttonIndex);
            Console.SetCursorPosition(40,buttonIndex);
            Console.CursorTop = buttonIndex;
            string keyName = operation.ShortCut == '\b' ? "BkSpc" : operation.ShortCut == '\r' ? "Enter" : operation.ShortCut.ToString();
            Console.Write("<" + keyName + ">\t" + operation.KeyText + "\t(" + operation.Name + ")");
        }
    }
```

The `DisplayButtons` method draws the right-hand side of the interface
listing the "buttons" and associated operations, by looping through the
list of operations provided by the engine.

```csharp
    static void WriteErrorLine(string errorString)
    {
        Console.SetCursorPosition(0, 0);
        Console.Write(errorString);
    }
```

The WriteErrorLine method just writes the given message in the error
line of the interface.

```csharp
    public static void Main()
    {
        rpnCalcEngine = new RPNCalculatorEngine();
        buttons = new Dictionary<char, int>();
        RPNCalculatorEngine.Operation operation;

        int i = 0;
        bool errorDisplayed = false;
        while ((operation = rpnCalcEngine.GetOperation(i)) != null)
        {
            buttons.Add(operation.ShortCut, i++);
        }
        DisplayButtons();
        do
        {
            DrawCalculator();
            int button;
            char keyChar = Console.ReadKey(true).KeyChar;
            if (keyChar == '\x1b') break;    // exit the loop if Escape key pressed
            if (buttons.TryGetValue(keyChar, out button))
            {
                try
                {
                    if (errorDisplayed)
                    {
                        Console.SetCursorPosition(0, 0);
                        WriteErrorLine("                                   ");
                        WriteErrorLine("                                   ");
                        DisplayButtons();
                        errorDisplayed = false;
                    }
                    rpnCalcEngine.PressKey(button);
                }
                catch (FormatException)
                {
                    WriteErrorLine("Invalid number format");
                    errorDisplayed = true;
                }
                catch (Exception e)
                {
                    WriteErrorLine(e.Message);
                    errorDisplayed = true;
                }
            }
        } while (true);
    }
}
```

The good old `Main` method.  Here we set up the calculator interface and
just run through the main application loop - reading keyboard input,
updating the calculator display, and displaying the error messages from
any exceptions thrown by the engine.

So that's it.  Next time I'll have a look at some of the things I've
been learning in ASP.Net MVC.
