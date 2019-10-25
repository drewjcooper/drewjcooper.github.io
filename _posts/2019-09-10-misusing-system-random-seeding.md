---
series: Misusing System.Random
title: Incorrect Seeding
category: Coding
---
{% include misusing-system-random-stats.md %}

Another error I saw in our coding test submissions was that of incorrectly
seeding the random number generator. Before I show you the coding issues I saw
in the test submissions, we need to understand what seeding actually does.
<!--more-->

### Deterministic Randomness

`System.Random` is a Pseudo-Random Number Generator (PRNG), meaning that it
generates what looks like a random series of numbers, but is actually a
deterministic sequence.  Being a deterministic sequence, it has to start from
somewhere, and that is what the seed does - defines the initial conditions of
the internal state machine that generates the number sequence.

The determinism can be seen by running the code below:

```csharp
var seed = 10;
var rnd1 = new Random(seed);
var rnd2 = new Random(seed);

var sequence1 = GetSequence(rnd1);
var sequence2 = GetSequence(rnd2);

Console.WriteLine($"{sequence1.SequenceEqual(sequence2)}");

List<int> GetSequence(Random rnd)
{
    return Enumerable.Range(0, 100).Select(_ => rnd.Next(1_000_000)).ToList();
}
```

If you copy and paste this code into [LINQPad](https://linqpad.net) then you'll
get the output "True", showing that the two sequences are equal.

### From whence the seed?

So now we have a chicken-and-egg problem. I want to use "random" numbers in my
dice rolling utility, and `System.Random` will generate these for me, but I need
a "random" seed so that I don't get the same sequence of dice rolls for each run
of the application. Where can I get that "random" seed?

{{ numbers[seed-constant] | capitalize }} candidates used a constant seed, like this:

```csharp
var random = new Random(0);
```

Obviously that's not going to work. Well it is, if you're happy getting the same
sequence of random numbers each time you run the program.

The {{ numbers[seed-poor] }} other candidates to use an explicit seed did it this way:

```csharp
for (var i = 0; i < 10; i++)
{
    new Thread(() =>
    {
        var rnd = new Random(DateTime.Now.Millisecond);

        // other stuff
    }).Start();
}
```

That's better. It'd be hard to predict what the value of the seed will be, but
there are only a thousand possible values, and thus a thousand possible
sequences of random numbers. That might be okay in some scenarios, but why limit
randomness when it's not necessary? A better source for a seed is
`Environment.TickCount`, which gives the number of milliseconds since the
computer last started. This property cycles through the full range if `Int32`
values every 49 days or so, assuming the machine stays up that long.

And it's `Environment.TickCount` that`System.Random`'s parameterless constructor
does. The code for the constructors looks like this:

```csharp
public Random()
    : this(Environment.TickCount)
{
}

public Random(int seed)
{
    // Bunch on stuff to setup the internal state of the PRNG.
}
```

### An Issue of Timing

So, when you need a single random number generator then using the parameterless
constructor is most likely the right way to go. However, there is another
problem in the multi-threaded code above, and that is one of timing.

> But see the update in the next post regarding .Net Core

Using a time-based seed source leads to the same seed being used multiple times
if the source is sampled more frequently than its temporal resolution.
`DateTime.Now.Millisecond` has a resolution of 1ms, but
`Environment.TickCount`'s resolution is only around 10-16ms.

When I tested the above code I consistently got results with 2 or 3 groups of
threads producing the same number sequence. Switching to `Environment.TickCount`
as the seed source led, more often than not, to all the threads producing the
same sequence of numbers. {{ numbers[seed-timing] | capitalize }}
{{ of-submissions }} got this wrong.

### The Solution

One obvious solution to the above timing problem is to delay instantiating the
`System.Random` instances relative to each other, so that they each get a
different key. This is a rather hacky solution, though.

One might also be tempted to use a single instance for the threads to share.
However, `System.Random` is not thread-safe, but that is a topic for another
post.

A better solution is to use one instance to seed the rest. You create a
root instance at the start of your application, and that becomes the source of
the seed for any new instances you need. So the above problematic code turns
into this:

```csharp
var rootRnd = new Random();

for (var i = 0; i < 10; i++)
{
    new Thread(seed =>
    {
        var rnd = new Random((int)seed);

        // other stuff
    }).Start(rootRnd.Next());
}
```

The next couple of posts will look at issues around the lifetime of
`System.Random` instances and multi-threading.
