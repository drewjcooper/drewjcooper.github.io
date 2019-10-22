---
series: Misusing System.Random
title: Improvements in .Net Core
category: coding
---
{% include misusing-system-random-stats.md %}

My discussion of `System.Random` pitfalls so far this series, and particularly
the [post on seeding][1], has been based on the [reference source for .Net
Framework][.Net].

I was looking at the the [source code for .Net Core 2.x][core], and noticed
something very interesting. The parameterless constructor no longer uses
`Environment.TickCount` as the seed value.

In fact, since v2.0; .Net Core has improved the construction of `System.Random`
to the point where, as long as you use the parameterless constructor, none of
the problems highlighted in the previous post exist. It just works. Let's see
how.

NOTE: .Net Core 1.x has the same construction code as .Net Framework, so has the
same timing problem with seeding.

## So what's new?

In .Net Core 2+, the code relevant to instantiation of `System.Random` looks like this:

```csharp
[ThreadStatic]
private static Random? t_threadRandom;
private static readonly Random s_globalRandom = new Random(GenerateGlobalSeed());

public Random()
    : this(GenerateSeed())
{
}

public Random(int seed)
{
    // Bunch of stuff to initialise internal state machine
}

/*=====================================GenerateSeed=====================================
**Returns: An integer that can be used as seed values for consecutively
            creating lots of instances on the same thread within a short period of time.
========================================================================================*/
private static int GenerateSeed()
{
    Random? rnd = t_threadRandom;
    if (rnd == null)
    {
        int seed;
        lock (s_globalRandom)
        {
            seed = s_globalRandom.Next();
        }
        rnd = new Random(seed);
        t_threadRandom = rnd;
    }
    return rnd.Next();
}

/*==================================GenerateGlobalSeed====================================
**Action:  Creates a number to use as global seed.
**Returns: An integer that is safe to use as seed values for thread-local seed generators.
==========================================================================================*/
private static unsafe int GenerateGlobalSeed()
{
    int result;
    Interop.GetRandomBytes((byte*)&result, sizeof(int));
    return result;
}
```

Let's unpack that:

* The `System.Random` type has a globally static instance of itself, that is
  initialised by using a cryptographically random integer form the BCrypt API as
  the seed.
* The parameterless constructor uses a thread-static instance of itself to get a
  random integer for it's seed. Thread-static instances are scoped to the
  thread, rather than the application domain, so each running thread gets its
  own instance.
* The first time the parameterless constructor is called on a given thread the
  thread-static instance will not exist, so it's constructed using a seed
  obtained from the globally static instance.
* The globally static instance is protected with a lock as `System.Random` is
  not thread-safe. More on that in the next post.

## But what does it mean?

This means that, in .Net Core, using the parameterless constructor gives the
right result, no matter when and where you do it. The seed is generated
randomly, rather than as a function of the the time the machine has been
running, which may have an element of predictability.

This also means that we no longer have the timing issues that I spoke about in
my previous [post on seeding][1]. We can quickly spin up multiple
instances of `System.Random` without fear that they'll use the same seed and,
therefore, produce the same sequence.

## Proving the case

I wrote a quick program to test this out. The code is [here][code-repo].

It's a console app which quickly creates 3 threads, which each create 3
instances of `System.Random`. That's 9 instances created in quick succession.
Each instance is sampled for 20 random values, and then the program dumps the
results. Here's an example of the results I saw:

```text
~\repos\drewjcooper.github.io.code\MultiInstanceSeeding> dotnet run -f netcoreapp1.1

  4.1: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  3.1: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  5.0: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  5.2: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  3.2: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  5.1: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  4.2: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  3.0: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60
  4.0: 70, 84, 69, 64, 35, 66, 91, 45, 52, 91, 40, 70, 14,  9, 98, 68, 61, 26, 96, 60

~\repos\drewjcooper.github.io.code\MultiInstanceSeeding> dotnet run -f netcoreapp2.0

  4.2:  4, 36, 17,  2,  8, 93, 75, 78, 63, 90, 22, 81, 20, 89, 34, 16, 62, 46, 46, 98
  3.2:  4, 40,  3, 30,  0,  3,  7, 31, 66, 77,  5, 34, 67, 69, 45, 67, 53, 13, 72, 80
  4.0: 37, 84, 52, 90, 22, 85, 71, 11, 42, 61, 17, 34, 85, 28, 77, 10, 25, 28, 68, 31
  3.0: 38, 67, 39, 10, 41, 94, 99, 55, 31, 77, 46, 89, 49, 60, 89, 36, 80, 74, 48, 17
  5.1: 39, 42, 36, 96, 57, 95, 51, 96,  4, 71, 24, 60, 90, 33, 83, 13, 67, 76, 44, 95
  3.1: 46, 69, 12, 39, 56, 35, 93, 29, 80, 14, 70, 83,  1, 10, 32, 78, 20,  9, 30, 53
  5.2: 49, 64, 72, 93, 98, 10, 90, 46, 32, 82, 75, 66, 20, 75, 31, 12, 31, 63, 30, 38
  4.1: 63, 74, 84, 49,  9,  9,  1, 58, 82, 90, 11, 55, 17, 70, 30,  1, 60, 16, 71, 85
  5.0: 85, 60, 84, 82, 44, 10, 52, 48, 62, 62, 60, 60, 52, 22, 72, 91, 56, 33, 24, 16
```

As you can see, all 9 instances produced the same sequence of random numbers
when running on .Net Core 1.1, but different sequences on .Net Core 2.0.

## Summary

When using .Net Core 2.0 and above, the timing problems associated with creating
multiple instances of `System.Random` disappear. In a multi-threaded
environment, having an instance per thread is important because `System.Random`
is not thread-safe. But more on that next time.

Using an [instance per invocation][2] is still not a great idea, but
at least we know that, in .Net Core, doing so in a tight loop will still give
reasonable results.

[core]: https://source.dot.net/#System.Private.CoreLib/shared/System/Random.cs
[.Net]: https://referencesource.microsoft.com/#mscorlib/system/random.cs
[code-repo]: https://github.com/drewjcooper/drewjcooper.github.io.code/tree/master/MultiInstanceSeeding
[1]: {% post_url 2019-09-10-misusing-system-random-seeding %}
[2]: {% post_url 2019-09-11-misusing-system-random-instance-per-invocation %}
