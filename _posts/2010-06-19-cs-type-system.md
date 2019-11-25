---
title: C#’s type system
tags:
- C# Specifics
- ".NET Specifics"
---
C# is a strongly-typed object-oriented language.  Many of the types will appear
familiar, at first glance, to those who know C or C++, and most C expressions
will behave similarly in C#, but the C# type-system is fundamentally different.

In my previous post I introduced C#'s integer types. In this post I'll look at
C#'s type system.
<!--more-->

All C# types derive from the [Common Type
System](https://docs.microsoft.com/en-us/dotnet/standard/base-types/common-type-system) (CTS) provided
by the .NET framework, and are inherited from the `System.Object` (keyword
`object`) class.  I'll look into the object class in more detail later, but it
means that all variables have some basic methods and properties.

[![Value Types
and Reference Types in the
CTS](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/media/index/value-reference-types-common-type-system.png
"Value Types and Reference Types in the
CTS")](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/index)

The diagram shows the inheritance hierarchy of types in the
CTS.

C# has two categories of types: ***value types*** and ***reference
types***.  Variables of value types directly contain their data, whereas
variable of reference types store a reference to a memory location
containing an object.  More details on the differences in a later post.

All value types are either struct types or enumeration types.  The types
we would normally think of as base types in C or C++ are actually
structs, and the keywords used to identify them are aliases for types in
the .NET framework.

The basic value types in C#, known as ***simple types***, are:

<table>
  <thead>
    <tr>
      <td colspan="2">Type category</td>
      <td>Reserved<br />word</td>
      <td>Aliased<br />type</td>
      <td>Literal<br />suffix</td>
      <td>Range of values</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="8">Integer<br />Types</td>
      <td rowspan="4">Signed</td>
      <td><code>sbyte</code></td><td><code>System.SByte</code></td><td></td><td>-128 … 127</td>
    </tr>
    <tr>
      <td><code>short</code></td><td><code>System.Int16</code></td><td></td><td>-32,768 … 32,767</td>
    </tr>
    <tr>
      <td><code>int</code></td><td><code>System.Int32</code></td><td></td><td>-2,147,483,648 … 2,147,483,647</td>
    </tr>
    <tr>
      <td><code>long</code></td><td><code>System.Int64</code></td><td><code>L</code></td><td>-9,223,372,036,854,775,808 … 9,223,372,036,854,775,807</td>
    </tr>
    <tr>
      <td rowspan="4">Unsigned</td>
      <td><code>byte</code></td><td><code>System.Byte</code></td><td></td><td>0 … 255</td>
    </tr>
    <tr>
      <td><code>ushort</code></td><td><code>System.UInt16</code></td><td></td><td>0 … 65,535</td>
    </tr>
    <tr>
      <td><code>uint</code></td><td><code>System.UInt32</code></td><td><code>U</code></td><td>0 … 4,294,967,295</td>
    </tr>
    <tr>
      <td><code>short</code></td><td><code>System.UInt64</code></td><td><code>UL</code></td><td>0 … 18,446,744,073,709,551,615</td>
    </tr>
    <tr>
      <td rowspan="2" colspan="2">Floating point<br />types</td>
      <td><code>float</code></td><td><code>System.Single</code></td><td><code>F</code></td><td>+/-1.5 x 10<sup>-45</sup> to 3.4 x 10<sup>38</sup>, 7-digit precision</td>
    </tr>
    <tr>
      <td><code>double</code></td><td><code>System.Double</code></td><td><code>D</code></td><td>+/-5.0 x 10<sup>-324</sup> to 1.7 x 10<sup>308</sup>, 15-digit precision</td>
    </tr>
    <tr>
      <td rowspan="3" colspan="2"></td>
      <td><code>decimal</code></td><td><code>System.Decimal</code></td><td><code>M</code></td><td>+/-1.0 x 10<sup>-28</sup> to 7.9 x 10<sup>28</sup>, 28-digit precision</td>
    </tr>
    <tr>
      <td><code>char</code></td><td><code>System.Char</code></td><td></td><td>Any Unicode character (UTF-16)</td>
    </tr>
    <tr>
      <td><code>bool</code></td><td><code>System.Boolean</code></td><td></td><td><code>true</code> or <code>false</code></td>
    </tr>
  </tbody>
</table>

The keywords and type names in the table above can be used interchangeably,
although as a matter of style it's probably a good idea to keep to one or the
other.

The suffices can be used to specify a numeric type when writing a literal number
in the code.

That's all I've got time for at the moment. I think I'll get into the second
exercise next time. See you then.
