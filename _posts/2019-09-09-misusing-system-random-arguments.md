---
title: Misusing System.Random
subtitle: Incorrect Arguments for `.Next(...)`
category: Coding
---
{% include misusing-system-random-posts.md %}

## {{ page.subtitle }}

The most basic error in using `System.Random` is to get the parameters for the
`Next` method wrong. Part of our coding tests involves generating a random key
value from `key0` to `key9`. Five of the ten submissions reviewed used the
following incorrect method call to get that random digit:

```csharp
rnd.Next(0, 9);
```

## What's the Problem?

You may have heard that there are 2 hard problems in Computer Science: naming
things, cache invalidation, and off-by-one errors. This is one or the latter.

The above call will return a number from 0 to 8, inclusive. That `9` in the
method call is the *exclusive* upper bound of the range for random number
generator. This is confusing if you just look at the parameter names (`maxValue`
in this case), but is clear in Microsoft's
[documentation](https://docs.microsoft.com/en-us/dotnet/api/system.random.next?view=netcore-2.2).

The `Random.Next` method has three overloads:

```csharp
int Next()
int Next(int maxValue)
int Next(int minValue, int maxValue)
```

The first two are equivalent to `Next(0, Int32.MaxValue)`, and `Next(0,
maxValue)`, respectively. The docs state that `minValue` is the "*inclusive* lower
bound" and `maxValue` is the "*exclusive* upper bound", so the numbers returned
will satisfy the relation: `minValue <= value < maxValue`.

## The Solution

When using `.Next` with range arguments you need to remember that the `maxValue`
parameter is the *exclusive* upper bound, so it needs to be one more than the
actual maximum value to want to produce.

So for generating our keys in the range of 0-9 inclusive we need either of:

```csharp
rnd.Next(10)
rnd.Next(0, 10)
```

To simulate a six-sided die you would do:

```csharp
rnd.Next(1, 7);
```

Next time I'll talk about pitfalls in seeding the random number generator.
