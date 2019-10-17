---
title: Misusing System.Random
series: Misusing System.Random
category: Coding
---
{% include misusing-system-random-posts.md %}

We've been interviewing for a new position at work recently, and we give all our
candidates a coding test to do, usually between the first and second interview.
The test involves design around caching of a key/value store in a multi-threaded
system. Part of the test requires generating a random set of keys for exercising
the cache. I've reviewed submissions from {{numbers[stats.submissions]}}
candidates so far and, to my surprise, not a single one used `System.Random`
correctly.

I was amazed at how little candidates for a mid to senior-level software
development position knew about generating random numbers, much less
multi-threading. I don't know if it says more about the candidates, or the
design of `System.Random`, but I thought I'd write a few posts to highlight the
problems and show the right way (or ways) to do things.

## How can I misuse thee? Let me count the ways

Here is a list of the misuse I've seen, and the number of interview candidates
of the {{numbers[stats.submissions]}} I've reviewed who misused `System.Random`
in their code in this way.

Misuse                               | Candidates
-------------------------------------|-------
Incorrect arguments for `.Next(...)` | {{arguments-bad}} ({{arguments-bad-pc}}%)
Incorrect seeding                    | {{seed-incorrect}} ({{seed-incorrect-pc}}%)
Singleton for multiple threads       | {{instance-singleton}} ({{instance-singleton-pc}}%)
Instance per invocation              | {{instance-per-invocation}} ({{instance-per-invocation-pc}}%)

The following snippet shows examples of all of these misuses of `System.Random`
in the context of generating random numbers from 0-9, inclusive.

```csharp
var rnd1 = new Random(0);  // Incorrect seeding

new Thread(DoStuff).Start();
new Thread(DoStuff).Start();

void DoStuff()
{
    // Random is not thread-safe
    var value1 = rnd1.Next(0, 9);  // Incorrect parameters

    var value2 = new Random().Next(10);  // Instance per invocation
}
```

## What's next?

In this blog post series I'm going to dive into each of these wrong ways of
using `System.Random`, explain why they're wrong, and show the correct usage.
I'll finish up with thoughts on abstracting random number generation to enable
thread-safety, dependency injection and unit testing.
