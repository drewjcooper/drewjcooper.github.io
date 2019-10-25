---
title: Exercise 6 (part 2) – When is a class not a class?
date: '2010-09-15T22:09:00.001+10:00'
tags: 
modified_time: '2010-09-15T22:09:43.487+10:00'
---
It's been a while, but I've finally found some time to post again.  In
my [last
post](http://learningcsharpnet.blogspot.com/2010/08/exercise-6-part-1-multi-file-dll.html)
I showed the basic structure of a DLL for the engine of an RPN
Calculator in response to [Prashant's 6th
exercise](http://www.jobsnake.com/seek/articles/index.cgi?openarticle&8533). 
In this post I'll start to look at the details.

[]{#more}

The first class I'll look at is not actually a class -- it's a struct. 
In C# **structs** are simliar to **classes**, but there are some very
important differences.  I'll look at te differences in more detail at
the end of the post, but for now lets look at the InputLine struct. 
This **struct** models the line on the calculator display that accepts
entry of numbers from the calculator keyboard.  Here's the code, with
some comments.

::: {.csharpcode}
       1:  namespace LearningCSharp

       2:  {

       3:      public partial class RPNCalculatorEngine

       4:      {

       5:          protected struct InputLine

       6:          {
:::

The declaration of a **struct** is similar to that of a **class**.  The
`protected` keyword limits access to this **struct** to the code of the
containing class, and any classes derived from it.

::: {.csharpcode}
       7:              private string _mantissa;

       8:              private bool _isNegative;

       9:              private string _exponent;

      10:              private bool _expIsNegative;

      11:              private bool _inExponent;

      12:              private bool _isActive;
:::

The content, and state of the input line are modelled by these fields.
Because the fields are declared private, they are only accessible within
the `InputLine` struct.

::: {.csharpcode}
      13:   

      14:              public bool IsActive { get { return _isActive; } }
:::

This code creates property called `IsActive`. This property allows code
using the struct to access the `_isActive` field.  Because the property
only defines a `get` accessor, the access in read-only.  Properties can
also define a set accessor to allow right access to the property.

::: {.csharpcode}
      15:   

      16:              public override string ToString()

      17:              {

      18:                  string mantissa = (_isNegative ? "-" : "") + _mantissa;

      19:                  string exponent = _inExponent ? "e" + (_expIsNegative ? "-" : "+") + _exponent : "";

      20:                  return mantissa + exponent;

      21:              }
:::

All classes and structs in C# ultimately derive from the `object`
class, so they all inherit the `ToString()` method.  The above code
overrides the method defined by `object` (hence the `override` keyword)
and returns a string representation of the content of the input line.

::: {.csharpcode}
      22:   

      23:              public double ToDouble()

      24:              {

      25:                  return double.Parse(this.ToString());

      26:              }
:::

The input line also needs to be accessed as a double so the calculator
can add it to the calculation stack.

::: {.csharpcode}
      27:              

      28:              public void Clear()

      29:              {

      30:                  _mantissa = null;

      31:                  _exponent = null;

      32:                  _isNegative = false;

      33:                  _expIsNegative = false;

      34:                  _inExponent = false;

      35:                  _isActive = false;

      36:              }
:::

The `Clear` method resets the input line to its default starting
values.  The `AddChar` method below is the core of the input line
struct.  It takes an input character and processes it, adding it to the
input string, or updating the state of the input line as needed.  It
returns `true` if the character is valid and was used, or `false`
otherwise.

::: {.csharpcode}
      37:   

      38:              public bool AddChar(char inputCharacter)

      39:              {

      40:                  if (inputCharacter >= '0' && inputCharacter <= '9')

      41:                  {

      42:                      if (_inExponent)

      43:                      {

      44:                          _exponent += inputCharacter;

      45:                      }

      46:                      else

      47:                      {

      48:                          _mantissa += inputCharacter;

      49:                      }

      50:                  }

      51:                  else

      52:                  {

      53:                      switch (inputCharacter)

      54:                      {

      55:                          case '.':

      56:                              if (!_inExponent && !_mantissa.Contains("."))

      57:                              {

      58:                                  _mantissa += inputCharacter;

      59:                              }

      60:                              break;

      61:   

      62:                          case '_':

      63:                              if (_inExponent)

      64:                              {

      65:                                  _expIsNegative = !_expIsNegative;

      66:                              }

      67:                              else if (_isActive)

      68:                              {

      69:                                  _isNegative = !_isNegative;

      70:                              }

      71:                              else

      72:                              {

      73:                                  return false;

      74:                              }

      75:                              break;

      76:   

      77:                          case 'E':

      78:                              _inExponent = true;

      79:                              break;

      80:   

      81:                          case '\b':

      82:                              if (_inExponent)

      83:                              {

      84:                                  if (_exponent.Length > 0)

      85:                                  {

      86:                                      _exponent = _exponent.Remove(_exponent.Length - 1);

      87:                                  }

      88:                                  else

      89:                                  {

      90:                                      _inExponent = false;

      91:                                  }

      92:                                  return true;

      93:                              }

      94:                              else if (_isActive)

      95:                              {

      96:                                  if (_mantissa.Length > 0)

      97:                                  {

      98:                                      _mantissa = _mantissa.Remove(_mantissa.Length - 1);

      99:                                  }

     100:                                  else

     101:                                  {

     102:                                      _isActive = false;

     103:                                      return false;

     104:                                  }

     105:                                  return true;

     106:                              }

     107:                              return false;

     108:                              

     109:                          default:

     110:                              return false;

     111:                      }

     112:                  }

     113:                  _isActive = true;

     114:                  return true;

     115:              }

     116:          }

     117:      }

     118:  }
:::

Now that I've briefly described the code of the `InputLine` struct , we
can look at the differences between a **struct** and a **class**. 

The main difference is that classes are reference types, allocated from
the heap, whereas structs are value types, allocated from the stack. 
Memory allocated from the heap is not initialised before it is used, but
memory allocated from the stack has all its bits initialised to 0.  For
this reason, when a struct is instantiated using the default
parameterless constructor all it's member fields are set to their
type-equivalent of 0.  Strings are set to null, numeric types set to 0,
boolean fields set to null, etc.

Another consequence of this difference is that you cannot inherit from a
struct, nor can a struct inherit from a reference type.  All structs
inherit from `System.ValueType`.

The other main difference is that, because structs already have a
parameterless constructor, you cannot declare a parameterless
constructor yourself.  Any constructors written for a struct must have
one or more parameters.

I think that will do it for this time.  Next post we'll look at the
classes that define the operations of the calculator and, as a
consequence, delegate types.
