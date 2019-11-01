---
title: Exercise 6 (part 1) – Multi-file DLL project
date: '2010-08-01T21:36:00.001+10:00'
tags:
- C# Specifics
- C# Exercise
modified_time: '2010-08-01T21:46:36.891+10:00'
thumbnail: http://lh3.ggpht.com/_8osJtRBSdMg/TFVcQzPCqTI/AAAAAAAAARE/SYPnqzpZcRI/s72-c/image_thumb17.png?imgmax=800
---
For his 6th exercise, [Prashant
suggests](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml)
writing a

> Scientific calculator supporting addition, subtraction, multiplication,
> division, square-root, square, cube, sin, cos, tan, Factorial, inverse,
> modulus.

In planning my solution to this exercise, I thought it would be cool to write a
DLL (a Class Library in .NET parlance) to be the core engine of the calculator,
and then write a console-based front-end.  Later, when I'm learning WPF I can
write a graphical front-end to the same DLL.
<!--more-->

So with that in mind, I set about designing the calculator engine.  I decided to
create an RPN calculator inspired by my HP 48GX.  I decided to write a class to
handle the input line, a group of classes to handle the various operations the
calculator can perform, and the engine class to tie it all together and provide
the interface used by the front-end. The diagram below shows the class
structure.

![RPN Calculator code layout](/assets/images/rpn-calculator-class-design.png)

As the diagram shows I also decided to split the project over multiple code
files.  This is very easy to do in C# and, as shown, even classes can be defined
in multiple files.

But first things first.  How is writing a DLL different to writing a console
application?  At the code level, not much.  The only difference in the code in
the absence of a `Main()` method.  The main difference is in the compiler.  The
**csc.exe** compiler has a command-line option call **/target**.  By default
this is set as /target:exe, which tells the compiler to produce a console
executable.  The **/target:winexe** option produces, you guessed it, a Windows
executable, and **/target:library** tells the compiler to create a DLL.  Of
course, if you're using Visual Studio, or Visual C# Express, you don't need to
worry about this - just specify Class Library as the project type when you're
creating a new project.

There's a couple of interesting things to note about the C# code when
writing a library over multiple code files, and in order to show you I
need to show you the basic skeleton of the code.  I'll cover the details
of the code in each file in the next few posts.  Here's the basic
skeleton:

### RPNCalculator.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace LearningCSharp
{
    public partial class RPNCalculatorEngine
    {
    }
}
```

### RPNCalculator.Operations.cs

```csharp
using System;
using System.Collections.Generic;

namespace LearningCSharp
{
    public partial class RPNCalculatorEngine
    {
        public delegate double BinaryOp(double operand1, double operand2);
        public delegate double UnaryOp(double operand);

        public class Operation
        {
        }

        public class BinaryOperation : Operation
        {
        }

        public class UnaryOperation : Operation
        {
        }

        public class ConstantOperation : Operation
        {
        }
    }
}
```

### RPNCalculator.InputLine.cs

```csharp
namespace LearningCSharp
{
    public partial class RPNCalculatorEngine
    {
        protected struct InputLine
        {
        }
    }
}
```

One thing that will immediately jump out to the C/C++ programmer is the complete
absence of header files. C# has no need of them. When compiling multiple files
like this into a single **assembly** (as an executable file - .exe or .dll - is
called in the .NET world), the compiler treats all the files together as one,
large code-space.  When referencing methods in external assemblies, the compiler
is able to look into the assembly at compile-time and make use of the meta-data
therein to hook it all together.

There are a few other things to notice in the code above, but I'll deal with just
two for the moment.  The rest I'll deal with when I cover the individual files
in detail.

The first is that the `RPNCalculatorEngine` class is defined in all three
files.  Notice, though, the use of the `partial` keyword in the class
declaration.   This keyword tells compiler that there may be code in other files
that goes to fully defining this class.  In every file in which the class is
declared the `partial` keyword must be used, and access modifier (in this case
`public`) must be the same.

The other thing to notice here is that most of the classes and other types are
nested in the `RPNCalculatorEngine` class.  I did this for two reasons - one,
because these types don't have much meaning outside the context of the
`RPNCalculatorEngine` class; and two, to show that it can be done.

Next time I'll dive into the details of the `InputLine` class.  Hope to see you
then.
