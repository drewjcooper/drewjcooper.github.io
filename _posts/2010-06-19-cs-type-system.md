---
title: C#’s type system
date: '2010-06-19T06:56:00.001+10:00'
tags:
- C# Specifics
- ".NET Specifics"
modified_time: '2010-06-19T06:58:02.584+10:00'
thumbnail: http://lh3.ggpht.com/_8osJtRBSdMg/TBvdfDUvKvI/AAAAAAAAAQ8/GihvbfBs6_8/s72-c/image%5B10%5D.png?imgmax=800
---
C# is a strongly-typed object-oriented language.  Many of the types
will appear familiar, at first glance, to those who know C or C++, and
most C expressions will behave similarly in C#, but the C# type-system
is fundamentally different.

In my previous post I introduced C#'s integral types.  In this post
I'll look at C#'s type system.

[]{#more}

All C# types derive from the [Common Type
System](http://msdn.microsoft.com/en-au/library/zcx1eb1e.aspx) (CTS)
provided by the .NET framework, and are inherited from the
**System.Object** (keyword `object`) class.  I'll look into the object
class in more detail later, but it means that all variables have some
basic methods and properties.  [![Value Types and Reference Types in the
CTS](http://lh3.ggpht.com/_8osJtRBSdMg/TBvdfDUvKvI/AAAAAAAAAQ8/GihvbfBs6_8/image%5B10%5D.png?imgmax=800 "Value Types and Reference
Types in the CTS"){width="293"
height="239"}](http://msdn.microsoft.com/en-au/library/ms173104.aspx "Types (C# Programming
Guide)")The diagram shows the inheritance hierarchy of types in the CTS.

C# has two categories of types: ***value types*** and ***reference
types***.  Variables of value types directly contain their data, whereas
variable of reference types store a reference to a memory location
containing an object.  More details on the differences in a later post.

All value types are either struct types or enumeration types.  The types
we would normally think of as base types in C or C++ are actually
structs, and the keywords used to identify them are aliases for types in
the .NET framework.

The basic value types in C#, known as ***simple types***, are:

::: {.scrollable}
Type category
:::

Reserved\
word

Aliased\
type

Literal\
suffix

Range of values

Integral\
Types

Signed

`sbyte`

System.SByte

 

-128...127

`short`

System.Int16

 

-32,768...32,767

`int`

System.Int32

 

-2,147,483,648...2,147,483,647

`long`

System.Int64

L

-9,223,372,036,854,775,808...9,223,372,036,854,775,807

Unsigned

`byte`

System.Byte

 

0...255

`ushort`

System.UInt16

 

0...65,535

`uint`

System.UInt32

U

0...4,294,967,295

`short`

System.UInt64

UL

0...18,446,744,073,709,551,615

Floating point\
types

`float`

System.Single

F

+/-1.5 x 10^-45^ to 3.4 x 10^38^, 7-digit precision

`double`

System.Double

D

+/-5.0 x 10^-324^ to 1.7 x 10^308^, 15-digit precision

 

`decimal`

System.Decimal

M

+/-1.0 x 10^-28^ to 7.9 x 10^28^, 28-digit precision

`char`

System.Char

 

Any Unicode character (UTF-16)

`bool`

System.Boolean

 

`true` or `false`

The keywords and type names in the table above can be used
interchangeably, although as a matter of style it's probably a good idea
to keep to one or the other.

The suffices can be used to specify a numeric type when writing a
literal number in the code.

That's all I've got time for at the moment.  I think I'll get into the
second exercise next time.  See you then.
