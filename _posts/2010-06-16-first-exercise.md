---
title: The First Exercise
tags:
- C# Exercise
last_modified_at: '2011-09-27T12:30:01.493+10:00'
---
The first item in Prashant's list of [15 Exercises for Learning a new
Programming
Language](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml)
is:

> Display series of numbers (1,2,3,4, 5\....etc) in an infinite loop.
> The program should quit if someone hits a specific key (Say ESCAPE
> key).

So this post we start to look at loops and integer types.
<!--more-->

Here's my solution to the exercise:

```csharp
using System;

class MainClass
{
    static void Main()
    {
        ulong lCount = 0;
        while (true)
        {
            Console.WriteLine((++lCount).ToString());
            if (Console.KeyAvailable && Console.ReadKey(true).KeyChar == '\x1B') break;
        }
    }
}
```

C# has 8 integer types:

Type    |Bits|                 Min Value|                 Max Value
--------|---:|-------------------------:|-------------------------:
`byte`  |   8|                         0|                       256
`sbyte` |   8|                      -128|                       127
`ushort`|  16|                         0|                    65,366
`short` |  16|                   -32,768|                    32,767
`uint`  |  32|                         0|             4,294,867,295
`int`   |  32|            -2,147,483,648|             2,147,483,647
`ulong` |  64|                         0|18,446,744,073,709,551,615
`long`  |  64|-9,223,372,036,854,775,808| 9,223,372,036,854,775,807

I've chosen to use a `ulong` to store the number we're going to
display, because it will give us the highest count.  Of course, you'd
have to wait a while before the count reached the limit of a short, but
what the heck.

Looping constructs in C# are, by-and-large, the same as in C and C++.
Here I've used `while (true) {}` to create an infinite loop.

In the next line we see something new.  In C# all types (with one
exception) are inherited from the `object` type.  Thus, even basic
integer types have a few basic methods, one of which is `ToString`.
Using this we can increment the counter, convert the result to a string,
and then write that string to the console.

The next line uses the Console class to check for the Escape key being
pressed.  `Console.KeyAvailable` returns true if there is a key
available in the keyboard buffer, that is, a key has been pressed.  The
`&&` operator ("and-also") only evaluates the right-hand expression if
the left-hand one is true.  So, if a key has been pressed we then get
the character associated with the key and compare it to the escape
character ('\x1b').

That'll do it for this post, I think.  Next time I'll have a look in
more detail into C#'s type system and play around a bit with the
integer types, and in the post after that we'll tackle Exercise 2.
