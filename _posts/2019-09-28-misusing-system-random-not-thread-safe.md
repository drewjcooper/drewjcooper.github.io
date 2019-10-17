---
series: Misusing System.Random
title: Singleton for Multiple Threads
category: Coding
toc: true
---
{% include misusing-system-random-posts.md %}

The last error I've seen in using `System.Random` is to use a single instance of
the class in an application that calls `.Next` in multiple threads.
{{numbers[instance-singleton] | capitalize}} {{of-submissions}} had this
problem, and, to be fair, so did I in my original response to our coding test.

[Microsoft's documentation][docs] says the following about using `System.Random`
across multiple threads:

>Instead of instantiating individual Random objects, we recommend that you
create a single Random instance to generate all the random numbers needed by
your app. However, Random objects are not thread safe. If your app calls Random
methods from multiple threads, you must use a synchronization object to ensure
that only one thread can access the random number generator at a time. If you
don't ensure that the Random object is accessed in a thread-safe way, calls to
methods that return random numbers return 0.

## Huh?

What was that? "calls to methods that return random numbers return 0"? That
sounds weird. And in running the code that our candidates had submitted, I have
never seen this behaviour.

To understand this statement I again dove into the [source code][source]. The
private method that is at the core of getting the next random number, and fields
it depends on, looks like this:

```csharp
private int _inext;
private int _inextp;
private readonly int[] _seedArray = new int[56];

private int InternalSample()
{
    int retVal;
    int locINext = _inext;
    int locINextp = _inextp;

    if (++locINext >= 56) locINext = 1;
    if (++locINextp >= 56) locINextp = 1;

    retVal = _seedArray[locINext] - _seedArray[locINextp];

    if (retVal == MBIG) retVal--;
    if (retVal < 0) retVal += MBIG;

    _seedArray[locINext] = retVal;

    _inext = locINext;
    _inextp = locINextp;

    return retVal;
}
```

`_seedArray` is an array of pseudo-random integers that is built from the seed
value in the constructor. `_inext` and `_inextp` are initially set to 0 and 21,
respectively. The seed array is circular, as the indices wrap around to 1 when
they reach the end.

So this code basically:

* maintains two indexes into the randomised array;
* increments both indexes;
* finds the absolute difference of the two indexed array elements;
* replaces one of the array values with this difference;
* and returns the difference value.

Looking at this carefully we can see race conditions with multiple threads
reading and writing the index fields, and also in the mutation of the seed
array. This may be complicated by the JIT optimiser reordering memory
operations. Looking at this code I hypothesised that, over time, and with enough
race collisions, the two indexes could converge on the same number. The result
would be `0` when taking the difference of the two indexed elements, and this
result would get written across the whole array over the next several calls to
`InternalSample()`. From there, it doesn't matter if the indexes diverge again.
The result of this method will always be `0`.

## Let's check it out

But to be sure, I put it to the test. I wrote a small console application that
samples `System.Random` in a tight loop on two threads. The code can be found
[here][code-repo].

The program tracks changes in the internal index fields and the difference
between them. If `System.Random` is behaving properly, with the indices being
incremented together, then the distance between them should remain constant.
Running the code, though, shows something different.

Here is a sample of the output of the program:

```text
Sample Count | Index Offset | inext | inextp
-------------|--------------|-------|-------
          11 |           21 |    11 |     32
     164,651 |           20 |    27 |     47
     196,772 |           19 |    26 |     45
     391,831 |           20 |    36 |      1
     414,271 |           19 |     1 |     20
     444,791 |           18 |    45 |      8
     687,552 |           17 |    35 |     52
     805,083 |           16 |    35 |     51
     998,891 |           15 |     6 |     21
   1,000,394 |           14 |    21 |     35
   1,032,161 |           13 |    50 |      8
   1,095,091 |           12 |    28 |     40
   1,215,291 |           10 |    53 |      8
   1,217,751 |            9 |    34 |     43
   1,321,631 |            8 |    38 |     46
   1,569,942 |            7 |    23 |     30
   1,663,211 |            6 |    53 |      4
   1,773,751 |            5 |     6 |     11
   2,154,352 |            4 |    22 |     26
   2,310,921 |            5 |     9 |     14
   2,434,362 |            4 |    19 |     23
   2,579,441 |            3 |    15 |     18
   2,638,602 |            2 |    55 |      2
   2,767,892 |            1 |    30 |     31
   2,774,282 |            0 |    26 |     26
```

The "Index Offset" column is the distance between the indices, moving to the
right around the circular array. You can see that the indices gradually converge
on the same value, at which point the seed array will fill with zeros and
subsequent call to `.Next()` will return 0.

The table shows the convergence happening after 2.7 million calls to `.Next`. In
my tests I've seen it occur in as little as 500,000 samples, or as many as 20-30
million.

So, don't use a single instance of `System.Random` in multiple threads.

## But I Need Randomness in Multiple Threads

Then use an instance of `System.Random` per thread. But there are potential
seeding issues as described in my [post on seeding][seeding]. If you're
targeting .Net Core 2.0 and above, [you're fine][core], and this is the simplest
solution.

However, if you're targeting .Net Framework, or .Net Core 1.x, then you're going
to need to do a bit more work. The simplest solution is to create a root
instance when you application starts, and use that to generate seeds for
per-thread instances. Something like this:

```csharp
class Program
{
    private static Random rootRandom = new Random();

    public static void Main(string[] args)
    {
        var thread = new Thread(DoSomething);
        thread.Start(rootRandom().Next());

        thread.Join();
    }

    private static void DoSomething(object seed)
    {
        var random = new Random((int)seed);

        // Do stuff with rnd
    }
}
```

Of course, you may want to encapsulate this logic in a separate class. Maybe
something like this:

```csharp
class ThreadSafeRandom
{
    private static Random rootRandom = new Random();
    [ThreadStatic]
    private static Random threadRandom;

    public ThreadSafeRandom()
    {
        if (threadRandom == null)
            lock(rootRandom)
                if (threadRandom == null)
                    threadRandom = new Random(rootRandom.Next());

        return threadRandom;
    }

    public int Next()
    {
        return threadRandom.Next();
    }
}
```

This implementation has the added benefit that if you create multiple instances
in a thread, then each instance will be using a single instance of
`System.Random` for the random numbers.

Another possibility is to have a a single instance of `System.Random`, but lock
it when sampling.

```csharp
class AnotherThreadSafeRandom
{
    private static Random random = new Random();

    public int Next()
    {
        lock(random)
        {
            return random.Next();
        }
    }
}
```

This ensures that only one thread can call `random.Next()` at a time, but at a
potential performance cost. You probably don't want to do this if you're
generating random numbers in a tight loop.

## Summary

You need to be careful when using `System.Random` in a multi-threaded
environment. Calls to the sampling methods are not thread-safe, and can corrupt
the internal state of the PRNG.

Use locks or separate instances for each thread (taking care to seed them
properly) to be safe.

[docs]: https://docs.microsoft.com/en-us/dotnet/api/system.random?view=netcore-2.2
[source]: https://source.dot.net/#System.Private.CoreLib/shared/System/Random.cs
[code-repo]: https://github.com/drewjcooper/drewjcooper.github.io.code/tree/master/SystemRandomIsNotThreadSafe
[seeding]: {% post_url 2019-09-10-misusing-system-random-seeding %}
[core]: {% post_url 2019-09-13-misusing-system-random-dotnet-core %}
