---
series: Misusing System.Random
title: Instance per Invocation
category: Coding
---
{% include misusing-system-random-stats.md %}

Another basic error in using `System.Random` is to use a new instance of the
class for each call to `.Next`.
<!--more-->

{{numbers[instance-per-invocation] | capitalize}} {{of-submissions}} did something equivalent to:

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

###Unnecessary Expense

So we have two basic problems. One is that constructing a new instance of
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

On my machine, running this code in [LINQPad](https://linqpad.net), the code
consistently outputs a number between 20 and 30. That is, it's 20-30 times more
expensive to create a new instance of `System.Random` for each call to `Next()`,
than to use a single instance for all calls. And that's not even considering the
GC pressure caused by all those extra objects.

###Pseudo-Non-Random Numbers

But that's not the only problem. Creating a new instance of `System.Random`
creates a new state machine for generating pseudo-random numbers. Initialising a
new state machine for each number generated compromises the randomness of the
values.

And from our previous discussion of the way that seeding works, it
should be clear that, if you're requesting multiple random numbers in a short
period of time, say rolling 5 dice for a game of Yahtzee, then there's a good
change that you'll end up with a run of the same number.

###Conclusion

So, don't create an instance of `System.Random` for each call to `Next()`.
Create a single instance once, then use it to generate all the random numbers
you need. Unless you're generating random numbers in multiple threads. But
that's the topic for next time.
