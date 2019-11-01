---
title: Exercise 6 (part 2) – When is a class not a class?
date: '2010-09-15T22:09:00.001+10:00'
tags: 
modified_time: '2010-09-15T22:09:43.487+10:00'
---
It's been a while, but I've finally found some time to post again.  In my [last
post]({% post_url 2010-08-01-exercise-6-part-1-multi-file-dll %})
I showed the basic structure of a DLL for the engine of an RPN Calculator in
response to [Prashant's 6th
exercise](https://www.articlecity.com/articles/computers_and_internet/article_2686.shtml).
In this post I'll start to look at the details.
<!--more-->

The first class I'll look at is not actually a class - it's a struct. In C#
**structs** are simliar to **classes**, but there are some very important
differences.  I'll look at te differences in more detail at the end of the post,
but for now lets look at the InputLine struct. This **struct** models the line
on the calculator display that accepts entry of numbers from the calculator
keyboard.  Here's the code, with some comments.

```csharp
namespace LearningCSharp
{
    public partial class RPNCalculatorEngine
    {
        protected struct InputLine
        {
```

The declaration of a **struct** is similar to that of a **class**.  The
`protected` keyword limits access to this **struct** to the code of the
containing class, and any classes derived from it.

```csharp
            private string _mantissa;
            private bool _isNegative;
            private string _exponent;
            private bool _expIsNegative;
            private bool _inExponent;
            private bool _isActive;
```

The content, and state of the input line are modelled by these fields.
Because the fields are declared private, they are only accessible within
the `InputLine` struct.

```csharp
            public bool IsActive { get { return _isActive; } }
```

This code creates a property called `IsActive`. This property allows code using
the struct to access the `_isActive` field.  Because the property only defines a
`get` accessor, the access in read-only.  Properties can also define a set
accessor to allow write access to the property.

```csharp
            public override string ToString()
            {
                string mantissa = (_isNegative ? "-" : "") + _mantissa;
                string exponent = _inExponent ? "e" + (_expIsNegative ? "-" : "+") + _exponent : "";
                return mantissa + exponent;
            }
```

All classes and structs in C# ultimately derive from the `object` class, so they
all inherit the `ToString()` method.  The above code overrides the method
defined by `object` (hence the `override` keyword) and returns a string
representation of the content of the input line.

```csharp
            public double ToDouble()
            {
                return double.Parse(this.ToString());
            }
```

The input line also needs to be accessed as a double so the calculator can add
it to the calculation stack.

```csharp
            public void Clear()
            {
                _mantissa = null;
                _exponent = null;
                _isNegative = false;
                _expIsNegative = false;
                _inExponent = false;
                _isActive = false;
            }
```

The `Clear` method resets the input line to its default starting values.  The
`AddChar` method below is the core of the input line struct. It takes an input
character and processes it, adding it to the input string, or updating the state
of the input line as needed.  It returns `true` if the character is valid and
was used, or `false` otherwise.

```csharp
            public bool AddChar(char inputCharacter)
            {
                if (inputCharacter >= '0' && inputCharacter <= '9')
                {
                    if (_inExponent)
                    {
                        _exponent += inputCharacter;
                    }
                    else
                    {
                        _mantissa += inputCharacter;
                    }
                }
                else
                {
                    switch (inputCharacter)
                    {
                        case '.':
                            if (!_inExponent && !_mantissa.Contains("."))
                            {
                                _mantissa += inputCharacter;
                            }
                            break;

                        case '_':
                            if (_inExponent)
                            {
                                _expIsNegative = !_expIsNegative;
                            }
                            else if (_isActive)
                            {
                                _isNegative = !_isNegative;
                            }
                            else
                            {
                                return false;
                            }
                            break;

                        case 'E':
                            _inExponent = true;
                            break;

                        case '\b':
                            if (_inExponent)
                            {
                                if (_exponent.Length > 0)
                                {
                                    _exponent = _exponent.Remove(_exponent.Length - 1);
                                }
                                else
                                {
                                    _inExponent = false;
                                }
                                return true;
                            }
                            else if (_isActive)
                            {
                                if (_mantissa.Length > 0)
                                {
                                    _mantissa = _mantissa.Remove(_mantissa.Length - 1);
                                }
                                else
                                {
                                    _isActive = false;
                                    return false;
                                }
                                return true;
                            }
                            return false;

                        default:
                            return false;
                    }
                }
                _isActive = true;
                return true;
            }
        }
    }
}
```

Now that I've briefly described the code of the `InputLine` struct, we
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
