---
title: Misusing System.Random
subtitle: Improvements in .Net Core
category: coding
---
{% include misusing-system-random-posts.md %}

My discussion of `System.Random` pitfalls so far this series, and particularly
the [post on seeding][1], has been based on the [reference source for .Net
Framework][.Net].

I was looking at the the [source code for .Net Core][core], and noticed
something very interesting. The parameterless constructor no longer uses
`Environment.TickCount` as the seed value.

In fact, .Net Core has improved the construction of `System.Random` to the point
where, as long as you use the parameterless constructor, none of the problems
highlighted in the previous post exist. It just works. Let's see how.

## So what's new?

In .Net Core, the code relevant to instantiation of `System.Random` looks like this:

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

Using an [instance per invocation][2] is still not a great idea, but
at least we know that, in .Net Core, doing so in a tight loop will still give
reasonable results.

[core]: https://source.dot.net/#System.Private.CoreLib/shared/System/Random.cs
[.Net]: https://referencesource.microsoft.com/#mscorlib/system/random.cs
[1]: {% post_url 2019-09-10-misusing-system-random-seeding %} 
[2]: {% post_url 2019-09-11-misusing-system-random-instance-per-invocation %}
