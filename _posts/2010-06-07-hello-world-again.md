---
title: Hello, World! – again
date: '2010-06-07T22:56:00.001+10:00'
tags:
- C# Exercise
modified_time: '2010-06-19T11:15:54.268+10:00'
---
In [15 Exercises for Learning a new Programming
Language](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml),
Prashant Mhatre kicks off with:

> First of all, get familiar with Compiler, compiler option, editor
> shortcuts or integrated development environment (IDE). Start with a
> simple 'Hello World' program. Compile it. Use basic functionalities of
> debugger like setting break points, printing variable values, moving
> to the next or specific position, stopping debugger etc.

Let's jump straight into the code first.
<!--more-->
Here's the most basic "Hello World" in C#:

```csharp
class MainClass
{
    static void Main()
    {
        System.Console.WriteLine("Hello, World!");
    }
}
```

This looks a little different to the way it's presented in most
tutorials, but I'll get to that in a minute.

The first thing to notice here is the **MainClass** class.  In general
terms, the only things that can be at the global level of a C# source
file are certain type declarations, namespace and alias declarations,
and using-namespace-directives.  This means that the entry point of the
program (the **main()** function in C) must actually be a method of a
class.  The name of the **class** here is not important. 

The **Main()** method must be a **static** member.  A static member
operates without reference to a specific class instance, and exists
independent of class instantiation.  More on this at a later time.

The **Main()** method, like in C, can be declared to return **void** or
**int**, and may take command-line arguments as an array of strings,
like this:

```csharp
static int Main(string[] args) { ... }
```

The last thing to notice about "Hello World" is the line that does the
actual work. In this line, **WriteLine** is a method of the **Console**
class which is a member of the **System** namespace.  In C#, classes,
and other types, are organised into a namespace hierarchy.  In the code
above the **MainClass** class is in the global namespace.  A program is
able to declare its own namespaces.

The other thing we can do is tell the compiler which namespaces we want
to use the members of, with a **using** directive.  So "Hello World" can
also be written:

```csharp
using System;

class MainClass
{
    static void Main()
    {
        Console.WriteLine("Hello, World!");
    }
}
```

This is the form usually seen in C# tutorials.  The first line says
that we want to be able to use the members of the System namespace
without using the fully qualified name.  Thus, in the program we are now
able to use the **Console** class directly.

Now that we've got some code we need a compiler so we can turn it into
an executable program.  The .NET framework provides a C# compiler
called **csc.exe**.  On my computer with version 4.0 of the .NET
framework, the compiler is at
C:\\Windows\\Microsoft.NET\\Framework\\v4.0.30319\\csc.exe.  I added the
folder to my PATH to make things easier.

Save the above code in a file called **HelloWorld.cs** and then do:

```console
csc.exe HelloWorld.cs
```

This will create HelloWorld.exe which will write the familiar greeting
to the console.  "csc.exe /?" will give the list of options available
when running the compiler.

Of course, an easier way to do things is to use an IDE, and for the
price (free) you really can't go past [Microsoft Visual C#
Express](http://www.microsoft.com/express/Downloads/).  I won't go into
all the options and debugging methods here -- I'll leave that as an
exercise for the reader.  Suffice to say that if you're at all familiar
with Microsoft's Visual Studio family of products you'll be right at
home.

I think that will do it for this time.  In the next post we'll tackle
Prashant's first exercise.
