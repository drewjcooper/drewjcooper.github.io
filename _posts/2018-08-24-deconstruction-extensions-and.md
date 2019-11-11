---
title: Deconstruction Extensions and Multitargeting
date: '2018-08-24T20:58:00.001+10:00'
tags:
- Deconstructors
- C#
modified_time: '2018-08-24T21:08:35.695+10:00'
---
In the [last post]({% post_url 2018-06-22-fun-with-deconstructors %}), I looked
at Deconstructors in C# 7.0. I thought I'd create a library with a few extension
methods to add Deconstructor support for Arrays and KeyValuePairs.
<!--more-->

I started with adding extension methods for arrays. I developed the methods
test-first and everything went pretty much as I expected it to. I ended up with
methods supporting deconstruction of arrays from 2 elements up to 9. Here's the
code for the 2-element method and its tests.

```csharp
public static class ArrayDeconstructors
{
    public static void Deconstruct<t>(this T[] array, out T item1, out T item2)
    {
        EnsureNotNull(array, nameof(array));
        EnsureMinimumLength(array, 2, nameof(array));

        item1 = array[0];
        item2 = array[1];
    }
}

public class TwoElementDeconstructorShould : ArrayDeconstructionTest
{
    public TwoElementDeconstructorShould() 
        : base(elementCount: 2)
    {
    }

    protected override void TestDeconstruction<T>(T[] array)
    {
        var (first, second) = array;

        first.Should().Be(array[0]);
        second.Should().Be(array[1]);
    }
}

public abstract class ArrayDeconstructionTest
{
    public ArrayDeconstructionTest(int elementCount)
    {
        ElementCount = elementCount;
    }

    public int ElementCount { get; }

    protected string ExpectedUndersizeArrayMessage =>
        $"The provided array must have at least {ElementCount} elements." +
        Environment.NewLine + "Parameter Name: array";

    [Fact]
    public void ThrowArgumentNullExceptionGivenNullInput()
    {
        int[] array = null;

        Action deconstruction = () => TestDeconstruction(array);

        deconstruction.Should().Throw<ArgumentNullException>().WithMessage("*array");
    }

    [Fact]
    public void ThrowArgumentExceptionGivenEmptyInt32Array()
    {
        var array = new int[0];

        Action deconstruction = () => TestDeconstruction(array);

        deconstruction.Should().Throw<ArgumentException>()
            .WithMessage(ExpectedUndersizeArrayMessage);
    }

    [Fact]
    public void ThrowArgumentExceptionGivenInt32ArrayWithInsufficientElements()
    {
        var array = new int[ElementCount - 1];

        Action deconstruction = () => TestDeconstruction(array);

        deconstruction.Should().Throw<ArgumentException>()
            .WithMessage(ExpectedUndersizeArrayMessage);
    }

    [Fact]
    public void DeconstructInt32ArrayOfMinimumSizeAsExpected()
    {
        var array = Enumerable.Range(1, ElementCount).ToArray();

        TestDeconstruction(array);
    }

    [Fact]
    public void ReturnFirstElementsOfLargerArray()
    {
        var array = Enumerable.Range(1, ElementCount).Reverse().ToArray();

        TestDeconstruction(array);
    }

    [Fact]
    public void DeconstructStringArrayAsExpected()
    {
        var array = new[] { "foo", "bar", "baz", "qux", "quux", "quuz", "corge", "grault", "garply", "waldo" };
        TestDeconstruction(array);
    }

    protected abstract void TestDeconstruction<T>(T[] array);
}
```

Things did not go as expected when I started on support for `KeyValuePair<K,V>`.
I'd created the project targeting .Net Standard 1.1, with a test project
targeting .Net Core 2.1. I wrote the first test for a `KeyValuePair<K,V>`.
Checking for null input is moot as KeyValuePair is a value type. So the first
test I wrote was for an actual deconstruction. Here it is:

```csharp
[Fact]
public void DeconstructKeyValuePairOfInt32Int32()
{
    var kvp = new KeyValuePair<int, int>(2, 4);

    var (key, value) = kvp;

    key.Should().Be(2);
    value.Should().Be(4);
}
```

Then I ran the test, expecting it to be red, actually expecting it to not even
compile when the compiler couldn't find a `Deconstruct` method. Imagine my
surprise when it not only compiled, but ran green! I had a passing test without
writing a line of code.

Turns out that a `Deconstruct` method was added to `KeyValuePair<K,V>`
in .Net Core 2.1. Here's what it looks like:

```csharp
[EditorBrowsable(EditorBrowsableState.Never)]
public void Deconstruct(out TKey key, out TValue value)
{
    key = Key;
    value = Value;
}
```

The attribute decorating it hides the method from Intellisense, but the compiler
knows it's there, and it just works.

So where to for my class library? I wanted the library to be able to be used in
projects targeting any platform, so .Net Standard 1.1 was the right choice
there. But a test project needs to target a concrete framework. I changed the
test project to target .Net Core 2.0 and my test failed. That is, it failed to
compile. I added a degenerate extension method so that the test would compile,
but still not pass. Then I modified the test project to target multiple
frameworks. In this case it was as simple as changing one line in the .csproj
file from

```xml
<TargetFramework>netcoreapp2.1</TargetFramework>
```

to

```xml
<TargetFrameworks>netcoreapp2.1;netcoreapp1.1;net452;net47</TargetFrameworks>
```

I then ran dotnet test and it compiled and ran the tests for each of the
targeted frameworks individually. The test passed for .Net Core 2.1, but failed
for the others. Cool. So then I was able to finish the extension method, and get
the class library to where I wanted it to be.

You can see the code on [GitHub](https://github.com/drewjcooper/deconstructors).
Next step is to publish a NuGet package just on the off chance that someone
might actually want to use it.
