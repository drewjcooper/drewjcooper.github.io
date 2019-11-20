---
title: Exercise 6 (part 3) – Delegates and Class Inheritance
tags:
- C# Specifics
- C# Exercise
- ".NET Specifics"
last_modified_at: '2011-04-11T11:28:06.748+10:00'
---
So, back to my RPN calculator implementation.  Bear in mind that I wrote
this code over six months ago, and I've learned a lot since then.  If I
was to do it again now I may make different design decisions, but for
the moment I'll stick with the original code.  Let's see what we can
learn.
<!--more-->

Last post I looked at the `InputLine` class.  A calculator also needs a set of
operations it can perform.  An operation can be an input operation, or can take
0, 1 or 2 operands from the operand stack (constant, unary or binary operation)
and returns a value to the stack. Each operation is bound to a key on the
calculator interface.

First things first, though.

```csharp
using System;
using System.Collections.Generic;

namespace LearningCSharp
{
    public partial class RPNCalculatorEngine
    {
        public delegate double BinaryOp(double operand1, double operand2);
        public delegate double UnaryOp(double operand);
```

I declare a couple of **delegate** types, `BinaryOp` and `UnaryOp`.  A
**delegate** is similar to a function pointer in C.  It basically gives you a
way to refer to a method and pass it around to get stuff done.  A variable of
type `BinaryOp `can refer to a method that takes two **double** parameters and
return a **double**.  A `UnaryOp `takes one **double** parameter and returns a
**double**.  We'll see these delegate types in use in a minute.

```csharp
        public class Operation
        {
            protected readonly string _name;
            protected readonly string _abbreviation;
            protected readonly string _keyText;
            protected readonly char _keyboardShortCut;
            protected readonly int _operandsRequired;

            public Operation(string name, string abbr, string keyText, char keyboardShortCut, int operandsRequired = 0)
            {
                _name = name;
                _abbreviation = abbr;
                _keyText = keyText;
                _keyboardShortCut = keyboardShortCut;
                _operandsRequired = operandsRequired;
            }

            public string Name { get { return _name; } }
            public string Abbreviation { get { return _abbreviation; } }
            public string KeyText { get { return _keyText; } }
            public char ShortCut { get { return _keyboardShortCut; } }
            public virtual bool HasFunction { get { return false; } }

            public virtual void Execute(Stack<double> operands) { }
        }
```

In this design, an **Operation** is something that happens based on user
interaction with the calculator interface.  Above is the `Operation` class,
which is the base class for all Operations.  The **class** includes read-only
**fields** for the name, abbreviation, text to display on calculator interface
(`_keyText`), keyboard shortcut to bind to the operation (`_keyboardShortCut`),
and the number of operands required, if any, for the operation.

Declaring a **field** as `readonly` means that it can only be set during object
instantiation, that is, in the constructor.  As there's no way to modify the
object once it's been created, an `Operation` object is immutable.  Declaring
the fields as `protected` means that they can only be accessed directly by code
within this class, or any class derived from this class.

The **class** also defines some public **properties** that give access to the
values stored in the fields to code outside of the class.  Note that
`_operandsRequired` doesn't have an associated **property**, so it's only
available within the class, or those derived from it.  Note also the
`HasFunction` property, which in this class returns `false`. The purpose of this
property is to indicate to client code whether or not this **operation** has a
function associated with it.  The **property** is declared `virtual` so that it
can be overridden by classes which derive from `Operation`.

Finally, the class has a **virtual method** called `Execute`, which in this case
does nothing.  In derived classes it will be overridden to execute the function
associated with the **operation**.

```csharp
        public class BinaryOperation : Operation
        {

            private readonly BinaryOp _function;

            public BinaryOperation(string name, string abbr, string keyText, char keyBoardShortCut, BinaryOp func)
                : base(name, abbr, keyText, keyBoardShortCut, 2)
            {
                _function = func;
            }

            public override bool HasFunction { get { return true; } }

            public override void Execute(Stack<double> operands)
            {
                if (operands.Count < _operandsRequired)
                {
                    throw new ArgumentException(_abbreviation + ": Insufficient operands");
                }
                operands.Push(_function(operands.Pop(), operands.Pop()));
            }
        }
```

Here's the definition of the `BinaryOperation` class.  Line 37 declares the
class to be derived from the `Operation` class (`: Operation`), so
`BinaryOperation` inherits all of `Operation`'s public and protected members
(fields, properties, methods, etc).

`BinaryOperation` adds a new field to store a function to be performed by the
operation.  Note that the type of this member is one of the **delegate** types
we declared earlier.  So, an object of the `BinaryOperation` class will store a
reference to a procedure that accepts two `double` parameters, and returns a
`double` result.

The constructor here takes a form that we haven't seen before.

```csharp
public BinaryOperation(string name, string abbr, string keyText, char keyBoardShortCut, BinaryOp func)
    : base(name, abbr, keyText, keyBoardShortCut, 2)
```

The constructor's signature is similar to that of the `Operation` constructor,
except that instead of a parameter for the number of operands it accepts a
delegate for the function to be associated with this operation.  The next line
defines the **constructor initialiser**. By default, all instance constructors
(except for `object`) call the parameterless constructor of the base class
(`base()`) before executing the body of the constructor.  This allows the base
class to handle the construction of its bit so the derived class doesn't have to
worry about it.  In this case we change the default behaviour by specifying the
**constructor initialiser** explicitly.  This basically passes most of the
parameters through to the `Operation` constructor, including the specification
of the number of operands needed for this operation.  The body of the
constructor then initialises the only thing specific to this class, the
`_function` field.

In order for the calculator to be able to make use of this function the
`BinaryOperator` class overrides the `Execute` method from `Operation`. The
calculator engine passes the operand stack to the Execute method. The method
checks that there are sufficient operands on the stack, and throws an exception
if not.  The exception includes an error message that can be used by the
calculator engine.

If there are enough operands on the stack, `Execute` pops the top two operands,
calls the function stored in the operation, and then pushes the result onto the
stack.  Pretty simple.

```csharp
        public class UnaryOperation : Operation
        {
            private readonly UnaryOp _function;

            public UnaryOperation(string name, string abbr, string keyText, char keyBoardShortCut, UnaryOp func)
                : base(name, abbr, keyText, keyBoardShortCut, 1)
            {
                _function = func;
            }

            public override bool HasFunction { get { return true; } }

            public override void Execute(Stack<double> operands)
            {
                if (operands.Count < _operandsRequired)
                {
                    throw new ArgumentException(_abbreviation + ": Insufficient operands");
                }
                operands.Push(_function(operands.Pop()));
            }
        }
```

The `UnaryOperation` class looks very similar to `BinaryOperation`.  The basic
differences are the delegate type passed to the constructor, the number of
required operands passed to the base constructor, and operation of the `Execute`
method.  In this class `Execute` pops a single operand off the stack and returns
the result of the function to the stack.

```csharp
        public class ConstantOperation : Operation
        {

            private readonly double _value;

            public ConstantOperation(string name, string abbr, string keyText, char keyBoardShortCut, double value)
                : base(name, abbr, keyText, keyBoardShortCut, 0)
            {
                _value = value;
            }

            public override bool HasFunction { get { return true; } }

            public override void Execute(Stack<double> operands)
            {
                operands.Push(_value);
            }
        }
    }
}
```

The `ConstantOperation` class is similar to the two above except that instead of
a function delegate, it stores a constant value. The `Execute` method just
pushes that value onto the operand stack.

So there we are, we've looked at the implementation of the Operations and the
InputLine for the calculator.  Next time we'll look at the rest of the
calculator engine and see how all this ties together.

Before I go, though, a couple of notes on the way this design would change if I
were doing it today. Since I designed these classes I've been learning a lot
about the .Net framework.  In the `System` namespace are a bunch of classes
which use generics to define easy to use delegates.  Instead of declaring the
`BinaryOp` and `UnaryOp` delegates we can use `Func<double, double, double>` and
`Func<double, double>`. And for consistency we could use `Func<double>` instead
of the value parameter for the `ConstantOperation` class.

See you next time.
