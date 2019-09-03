---
title: Misusing System.Random - Arguments and Seeds
category: Coding
---
This is the second post in [a series]({% post_url
2019-09-03-misusing-system-random %}) on common ways that `System.Random` is
misused. The series is based on my experience reviewing code test submissions
from job candidates. 

# Incorrect arguments for `.Next(...)`

The most basic error in using `System.Random` is to get the parameters for the
`Next` method wrong. Part of our coding tests involves generating a random key
value from `key0` to `key9`. Five of the nine submissions reviewed (over half)
used the following incorrect method call to get that random digit:

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


# Seeding

