---
title: Misusing System.Random - Arguments and Seeds
category: Coding
---

# Incorrect parameters

The most basic error in using `System.Random` is to get the parameters for the
`Next` method wrong. Part of our coding tests involves generating a random key
value from `key0` to `key9`. Five of the nine submissions (over half) used the
following incorrect method call to get that random digit:

```csharp
rnd.Next(0, 9);
```

This call will return a number from 0 to 8, inclusive. That `9` in the method
call is the *exclusive* upper bound of the range for random number generator.
This is confusing if you just look at the parameter names (`maxValue` in this case), but is clear in Microsoft's
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

In the case of generating our random keys, the correct usage is then
`rnd.Next(10)` or `rnd.Next(0, 10)`. As mentioned above, less than half our
submissions got this right.

# Instance per invocation

Another basic error in using `System.Random` is to use a new instance of the
class for each call to `.Next`. Two of the nine submissions did something
equivalent to:

```csharp
var value = new Random().Next(0, 10);
```

This avoids any issues with thread-safety, or lack thereof (more on that later),
in `System.Random`, but there are a couple of reasons why this is bad.

First of all, `System.Random`, is a Psuedo-Random Number Generator, or PRNG. It
generates "random" numbers using a deterministic algorithm.  The number sequence
appears to be random, following a uniform distribution, but is actually
generated from a kind of algorithmic state machine. The constructor does a bunch
of work to set up the state machine, and then each call to `Next()` gets a value
and increments the state of the machine.  

So we have two basic problems.  One is that constructing a new instance of
`System.Random` is relatively expensive. The following code is a basic benchmark
comparing the cost of generating a million random numbers from a single
instance vs. creating a new instance for each call to `Next()`.

```csharp
var sw = new Stopwatch();
var iterations = 1_000_000;

var rnd1 = new Random();
sw.Start();
for (int i = 0; i < iterations; i++)
{
	rnd1.Next(0, 10);
}
sw.Stop();
var singleInstanceDuration = sw.ElapsedMilliseconds;

sw.Restart();
for (int i = 0; i < iterations; i++)
{
	var rnd2 = new Random();
	rnd2.Next(0, 10);
}
sw.Stop();
var instancePerNextDuration = sw.ElapsedMilliseconds;

Console.WriteLine(instancePerNextDuration / singleInstanceDuration);
```

On my machine, running this code in LINQPad, the code consistently outputs a
number between 20 and 30. That is, it's 20-30 times more expensive to create a
new instance of `System.Random` for each call to `Next()`, than to use a single
instance for all calls.

But that's not the only problem. Creating a new instance of `System.Random`
creates a new state machine for generating pseudo-random numbers. Initialising a
new state machine for each number generated compromises the randomness of the
values. This is exacerbated by the way the state machine is initialised, but
we'll talk more about that in the next post about seeding.

For now, the takeaway is, don't create an instance of `System.Random` for each
call to `Next()`. Create a single instance once, then use it to generate all the
random numbers you need.

# Seeding


# Not thread safe


# The right way
# Testing non-deterministic code deterministically
