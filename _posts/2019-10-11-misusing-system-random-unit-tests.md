---
series: Misusing System.Random
title: Unit Testing Randomness
category: Coding
---
{% include misusing-system-random-stats.md %}

The previous posts in this series detailed a number of ways that I had seen
`System.Random` be misused in responses to a coding challenge we ask prospective
employees to complete, and described ways to avoid these pitfalls.
<!--more-->

There is another issue to consider when using `System.Random`, and that is
testability of code. This is not just a problem with `System.Random`, of course.
It's an issue of writing deterministic test in any context which includes a
non-deterministic component. The same problem occurs with code that gets the
current date or time, or makes a network or database call, or even reads or
writes a file system.

## The Problem

Let say you're writing an engine for your favourite role-playing game. You will,
of course, need a way to roll various combinations of dice, so you write this
simple class:

```csharp
class Dice
{
    private Random random = new Random();

    public int Roll(int quantity, int sides, int modifier)
    {
        var total = modifier;

        for (int i = 0; i < number; i++)
        {
            total += random.Next(sides) + 1;
        }

        return total;
    }
}
```

So calling `dice.Roll(2, 6, 3)` is the equivalent of rolling `2d6+3`.

But how would you unit test this? Given those 3 parameters there are 11 possible
outcomes ranging from 5 to 15. The best you could do is to assert that the
result is in the valid range, or even run a number of samples and assert the
expected distribution.

The former says nothing about whether the results are
being calculated correctly; a method that returns `sides + modifier` would pass
the test for any set of parameters.

The latter option would take longer to run, and would be fragile and error
prone. Due to the nature of randomness there would have to be some fuzziness in
the distribution you're asserting, but how fuzzy can you get before the test
becomes meaningless?

We're familiar with the idea of developing and testing clients without the
non-determinism and latency of network or database calls by mocking the interface
that those calls are made through. Can we use the same technique here?

## The Solution

`System.Random` does not implement an interface, so our best option, short of
creating a shim using a library like Microsoft Fakes (which is problematic for a
number of reasons), is to wrap it in a small wrapper class that does. The added
benefit of this approach is that the wrapper interface only needs to expose the
methods we need, thus making it easier to mock.

So we can do something like this:

```csharp
interface IRandomNumberGenerator
{
    int Next(int exclusiveMaximum);
}

class RandomNumberGenerator : IRandomNumberGenerator
{
    private Random random = new Random();

    int Next(int exclusiveMaximum)
    {
        return random.Next();
    }
}
```

With an interface and a wrapper like this in place, we can now modify our `Dice`
class.

```csharp
class Dice
{
    private IRandomNumberGenerator rng;

    public Dice(IRandomNumberGenerator rng)
    {
        this.rng = rng
    }

    public int Roll(int quantity, int sides, int modifier)
    {
        var total = modifier;

        for (int i = 0; i < quantity; i++)
        {
            total += rng.Next(sides) + 1;
        }

        return total;
    }
}
```

And now we've got something that's testable. I can create a simple fake that
takes a sequence of numbers and returns subsequent elements of that sequence on
each call to `Next`.

```csharp
class FakeRandomNumberGenerator : IRandomNumberGenerator
{
    private IEnumerator<int> enumerator;
    private int expectedParameter;

    public FakeRandomNumberGenerator(IEnumerable<int> sequence)
    {
        enumerator = sequence.AsEnumerable().GetEnumerator();
    }

    public FakeRandomNumberGenerator ExpectNext(int expectedParameter)
    {
        this.expectedParameter = expectedParameter;
        return this;
    }

    public int Next(int exclusiveMaximum)
    {
        if (exclusiveMaximum != expectedParameter)
        {
            throw new Exception($"Expected call to .Next({expectedParameter}), not .Next({exclusiveMaximum}).");
        }

        return enumerator.MoveNext() ?
            enumerator.Current :
            throw new Exception("No more numbers in the sequence");
    }
}
```

I'm throwing an exception here if the test samples more values that it provides,
but the fake could be written to behave however you'd like. You could also
implement something like this using your favourite mocking library.

Now we can write deterministic tests for our `Dice` class.

```csharp
[TestCase(6, new[] { 5, 2, 4 }, 3)]
[TestCase(4, new[] { 1, 3, 2, 4 }, 6)]
public void Roll_ReturnsSumOfDicePlusModifier(
    int sides,
    IEnumerable<int> rolls,
    int modifier)
{
    var expected = rolls.Sum() + modifier;
    var randomSequence = role.Select(r => r - 1); // 0 to sides - 1
    var rnd = new FakeRandomNumberGenerator(randomSequence)
        .ExpectNext(sideCount);
    var sut = new Dice(rnd);

    var result = sut.Roll(rolls.Count(), sides, modifier);

    result.Should().Be(expected);
}
```

Here I've created test cases for `3d6+3` and `4d4+6`. The test cases are defined
in terms of the actual die rolls (1 to number-of-sides), but the random number
generator gives values from 0 to max-1, so I've adjusted the sequence before
telling the `FakeRandomNumberGenerator` what to produce. You could define the
test case in terms of the random sequence, but then you'd have to modify the
calculation of the expectation.

## Summary

When writing unit tests for code that calls `System.Random`, it's good to be
able to replace the instance of `System.Random` with a mock or fake allowing you
to control the sequence of numbers produced. This gives you control over test
cases, allowing you to easily test edges cases, and makes you tests
deterministic and much easier to reason about.

The same technique can be used to enable deterministic testing of code that
calls `System.DateTime.Now`, or makes use of values from any non-deterministic
source.

That's it for my series on misusing `System.Random`. Not sure what's coming up
next, but please stay tuned.
