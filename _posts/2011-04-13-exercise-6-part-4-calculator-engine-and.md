---
title: Exercise 6 (part 4) – The Calculator Engine and Anonymous Methods
tags:
- C# Specifics
- C# Exercise
- .NET Specifics
---
In this post I'll complete the RPN Calculator engine we've been looking
at.  This will complete the DLL I've been working on, and next time I'll
run through the code for a Console-based front-end.
<!--more-->

Let's get straight into the code.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace LearningCSharp
{
    public partial class RPNCalculatorEngine
    {
        private Stack<double> _calculationStack;
        private InputLine _input;
        private List<Operation> _operations;
        static private int _baseOperationCount = 30;
```

The `RPNCalculatorEngine` class defines four **private fields**: one for the
operand stack, one for the input line, one for the list of operations supported
by the engine, and one for the number of operations supported by the engine.
The `_calculationStack` and `_operations` fields are declared as generic types
from the `System.Collections.Generic` **namespace**.  Note that the
`_operations` field is a `List` of `Operation`.  We're actually going to use it
to store objects of `Operation`, `BinaryOperation`, `UnaryOperation` and
`ConstantOperation` types, but that's okay because the last three are derived
from Operation as we saw in the last post.

Next, let's have a look at the constructor:

```csharp
        public RPNCalculatorEngine()
        {
            _calculationStack = new Stack<double>();
            _input = new InputLine();
            _input.Clear();
            _operations = new List<Operation>(_baseOperationCount);
            _operations.Add(new Operation("Zero", "0", "0", '0'));
            _operations.Add(new Operation("One", "1", "1", '1'));
            _operations.Add(new Operation("Two", "2", "2", '2'));
            _operations.Add(new Operation("Three", "3", "3", '3'));
            _operations.Add(new Operation("Four", "4", "4", '4'));
            _operations.Add(new Operation("Five", "5", "5", '5'));
            _operations.Add(new Operation("Six", "6", "6", '6'));
            _operations.Add(new Operation("Seven", "7", "7", '7'));
            _operations.Add(new Operation("Eight", "8", "8", '8'));
            _operations.Add(new Operation("Nine", "9", "9", '9'));
            _operations.Add(new Operation("Decimal", ".", ".", '.'));
            _operations.Add(new Operation("Exponent", "Exp", "E", 'E'));
            _operations.Add(new Operation("Back", "Bck", "Back", '\b'));
            _operations.Add(new UnaryOperation("Enter", "Cpy", "Enter", '\r', Copy));
            _operations.Add(new UnaryOperation("Negate", "Neg", "+/-", '_', x => -x));
            _operations.Add(new UnaryOperation("Inverse", "Inv", "1/x", '\\', x => 1 / x));
            _operations.Add(new ConstantOperation("PI", "PI", "\x03c0", 'p', Math.PI));
            _operations.Add(new ConstantOperation("e", "e", "e", 'e', Math.E));
            _operations.Add(new BinaryOperation("Add", "Add", "+", '+', (y, x) => x + y));
            _operations.Add(new BinaryOperation("Subtract", "Sub", "-", '-', (y, x) => x - y));
            _operations.Add(new BinaryOperation("Multiply", "Mul", "x", '*', (y, x) => x * y));
            _operations.Add(new BinaryOperation("Divide", "Div", "\xf7", '/', (y, x) => x / y));
            _operations.Add(new BinaryOperation("Modulus", "Mod", "mod", '%', (y, x) => x % y));
            _operations.Add(new UnaryOperation("Factorial", "Fac", "!", '!', Factorial));
            _operations.Add(new UnaryOperation("Cosine", "Cos", "cos", 'c', Math.Cos));
            _operations.Add(new UnaryOperation("Sine", "Sin", "sin", 's', Math.Sin));
            _operations.Add(new UnaryOperation("Tangent", "Tan", "tan", 't', Math.Tan));
            _operations.Add(new UnaryOperation("Square", "x\xb2", "x\xb2", 'S', x => Math.Pow(x, 2)));
            _operations.Add(new UnaryOperation("Cube", "x\xb3", "x\xb3", 'C', x => Math.Pow(x, 3)));
            _operations.Add(new UnaryOperation("SquareRoot", "RtX", "\x221a", 'R', Math.Sqrt));
        }
```

So that all looks a little over the top, and looking back on it now I can think
of a number of better ways of doing this, but basically all we're doing is
initialising the operand stack and the input line, and populating the list of
built-in operations.  I say "built-in", because, although I haven't written a
method to enable it, with this design it's possible for the calculator front-end
that uses this DLL to extend the list of operations the calculator can perform.

There are two interesting points here. One is the mixing of operation types in
the one list that I mentioned before. It's just classic OO polymorphism, but
it's interesting to see it in action.

The more important thing to notice, from the point of view of learning C#, are
the way that the functions are passed to the operation constructors. Take the
**Add** operation, for example.  The function passed to the constructor is `(x,
y) => x + y`.  At first that looks more than a little strange, but it's called a
**lambda expression**, and it's basically just a notation for defining
anonymous, or inline, functions.  Basically it defines a function that takes two
operands and returns their sum.  In this context at least, we don't need to
declare the types of the operands, because the compiler can infer them from the
context in which we're using the **lambda**.  The lambda expression is mapped to
the **delegate** type we used to declare the constructor parameter (`BinaryO
p`), and so the operands are taken to be two doubles, and the result their sum.

We also see a different application of the **delegate** parameter here. The trig
operations (sin, cos and tan) are not defined in terms of **lambda**s.  We don't
need an anonymous function because we already have perfectly suitable methods
defined in the `Math` class.  Instead of a **lambda** we just pass in a
reference to the appropriate method. As long as the signature of the method
matches the signature of the **delegate** type, we're fine.

One other point of interest is the use of Unicode character codes in some of the
strings.  In C#, all `string`s and `char`s, are defined in terms of Unicode
characters.  Any Unicode character can be used in a `char` or `string` literal
by means of the `\x…` escape code.

```csharp
        public string InputString
        {
            get
            {
                return this._input.ToString();
            }
        }
```

Here we have a read-only property to get a string representation of the
current input line.  Note that the property has a getter, but not
setter.

```csharp
        public void PressKey(int keyIndex)
        {
            if (keyIndex < _operations.Count)
            {
                Operation operation = _operations[keyIndex];
                if (_input.AddChar(operation.ShortCut)) return;
                if (operation.ShortCut == '\b' && _calculationStack.Count > 0)
                {
                    _calculationStack.Pop();
                    return;
                }
                if (operation.HasFunction)
                {
                    if (_input.IsActive)
                    {
                        _calculationStack.Push(_input.ToDouble());
                        _input.Clear();
                        if (operation.Name == "Enter") return;
                    }
                    operation.Execute(_calculationStack);
                }
            }
        }
```

The `PressKey` method is basically the core of the engine.  It is the main
method called by the calculator UI to pass user key strokes to the engine. The
interface is defined in terms of a number of keys, each of which map to one of
the operations in the calculator engine.  The parameter passed to this method is
the numeric index of the key that was pressed (operation to be  executed).

The `keyIndex` is check to see if it's in the correct range, and then operation
associated with the key is retrieved.  The character associated with the
operation is passed to the `inputLine`, and if it was a valid input character
(`_input.AddChar()` returns true) we finish processing the keystroke.  If the
key pressed was the back-space key then we just pop the top operand from the
stack and return.

Otherwise we check if the operation has a function defined.  If so, we check if
the `inputLine` is active and push it to the stack if it is. We then execute the
operation.  That's all there is to it.  As we saw last post, the operation pops
its operands from the stack and returns its result to the stack.

```csharp
        public double? this[int index]
        {
            get
            {
                if (index < _calculationStack.Count)
                    return _calculationStack.ElementAt<double>(index);
                else
                    return null;
            }
        }
```

Here are a couple of new things we haven't seen before.  The above code is an
**indexer**.  We'll see how the indexer is called next post when we look at the
calculator UI.  For now it's sufficient to say that it behaves a little like a
property, but with an index parameter.  This indexer only has a getter, so it's
read only, and it returns the value of the appropriate position on the operand
stack, or null if the index is out of range.

In order to be able to return null, the return type is declared as`double?`.
This is the same as Nullable\<double\>, and all value types in C# have a similar
nullable variant.  A nullable type has two important properties -- `HasValue`
and `Value`.  The former returns **true** if a value is assigned, or false if
the variable is null.  The latter returns the value that has been assigned.

```csharp
        public Operation GetOperation(int index)
        {
            return index < _operations.Count ? _operations[index] : null;
        }
```

`GetOperation` just returns a reference to the operation with the specified
index in the operations list.  It probably would have made sense to implement
this as an indexer, where the index parameter was the character code associated
with the operation.  I can't remember now why I didn't end up doing it that way.

```csharp
        private double Copy(double operand)
        {
            _calculationStack.Push(operand);
            return operand;
        }
```

The last two members of this class are a couple of methods to implement
operation functions that we can't handle just with a **lambda**.  
`Copy` pushes a copy of it's parameter onto the stack, and then returns then
same value.  Recall that the `Operation` class pushes this return value onto the
stack.  The effect of this is that the top entry on the stack is copied.

```csharp
        private double Factorial(double operand)
        {
            if (operand >= 0d && Math.Floor(operand) == operand)    // Check that operand is a positive integer
            {
                double result = 1;
                while (operand > 1)
                {
                    result *= operand--;
                    if (double.IsInfinity(result))
                    {
                        throw new OverflowException("Fac: Result too large");
                    }
                }
                return result;
            }
            else
            {
                _calculationStack.Push(operand);    // Return the operand to the stack
                throw new ArgumentOutOfRangeException("Fac: Must be positive integer");
            }
        }
    }
}
```

This last method implements a basic factorial calculation.  If the operand is
negative, the factorial operation is undefined, so we just push the operand back
onto the stack and throw and **exception**.

We've seen in a number of places in this post, the use of members of the `Math`
class.   This is a class of static methods implementing a fairly wide range of
standard mathematical operations.

So that ties up the RPN Calculator Engine DLL.  Next time we'll have a look at a
simple console-based user interface for the calculator, and look at some pretty
cool capabilities of the `Console` class.

See you then.
