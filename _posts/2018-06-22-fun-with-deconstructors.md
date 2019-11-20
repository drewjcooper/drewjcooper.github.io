---
title: Fun with Deconstructors
tags:
- Deconstructors
- C#
last_modified_at: '2018-08-24T21:11:59.081+10:00'
---
In my [last post]({% post_url 2018-06-16-fun-with-initialisers %}) I had some
fun with C# initialisers. This time we'll look at Deconstructors that were
introduced as part of the new [`ValueTuple` support in C#
7.0](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7#tuples).
The code we'll look at this time will probably be slightly more useful for
real-world production code.
<!--more-->

### The basics

I won't go into a great deal of detail on the basics here because it's already
well covered in the above Microsoft docs link, and in another [blog
post](https://andrewlock.net/deconstructors-for-non-tuple-types-in-c-7-0/) about
support for deconstructors in custom types. Suffice to say that `ValueTuple`
types, and custom types with an appropriate `Deconstruct()` method, can be
deconstructed, or broken apart, to multiple variables in a single assignment
statement. So you can do stuff like:

```csharp
var point = (1.2, 5.3);
var (x, y) = point;

var customer = GetCustomer();
var (firstName, lastName) = customer;
```

### Beyond the basics

Deconstructors are also supported for types you can't change by means of
extension methods. So, we could deconstruct a `DateTime`, for example.

```csharp
public static void
Deconstruct(this DateTime value, out int year, out int month, out int day)
{
    year = value.Year;
    month = value.Month;
    day =
value.Day;
}

var (year, month, day) = DateTime.Today;
Console.WriteLine($"Today is {year}/{month}/{day}");
```

That's cool, but you've probably read about that usage in a number of places.
What I really want to look at, though, is what happens when you apply
deconstruction to arrays.

### Deconstructing arrays

You've probably seen, or written, code like this:

```csharp
string[] nameComponents = customer.Name.Split(' ');
string firstName = nameComponents[0];
string lastName = nameComponents[1];

// Do something with firstName and lastName
```

In this code I deliberately left the types explicit, rather than using `var`, to
make them clear.

If we define an appropriate extension method we can greatly simplify this code
by removing a bunch of details that just get in the way of readability.

```csharp
public static void Deconstruct(
    this string[] array, out string value1, out string value2)
{
    value1 = array[0];
    value2 = array[1];
}

var (firstName, lastName) = customer.Name.Split(' ');
//Do something with firstName and lastName
```

Notice that the `Deconstruct` method only cares about the type of the array and
the return (`out`) parameters. You can create multiple methods with different
numbers of `out` parameters for arrays of different sizes. The method to use is
chosen by the number of variables on the left side on your deconstruction
assignment statement. Obviously you need to be careful that the array you're
deconstructing actually has sufficient elements.

You can see that this technique has immediate application to string parsing, but
it could probably be useful for other scenarios as well, and also for arrays of
other types. You don't need to define separate methods for different array
types, however. This technique can also be used with generics:

```csharp
public static void Deconstruct<T>(this T[] array, out T value1, out T value2)
{
    value1 = array[0];
    value2 = array[1];
}
```

Using this method you can now pull the first two elements out of an array of any
type.

Pretty cool, huh? But we don't have to stop at arrays. Another area where
assignments and ceremony get in the way is in iterating over dictionaries.

### Deconstructing dictionaries

Okay, so we're not actually going to apply deconstruction to the dictionary
itself, but deconstruction can be useful to improve readability when iterating
over dictionaries. Before we get to that, though, let's look at how we typically
enumerate a dictionary.

Let's say we're using a dictionary to keep a count of the occurrences of words
in a block of text. We then want to report those results.

```csharp
Dictionary<string, int> wordCounts = CountWords(text);
```

When we want to report these results we can enumerate the dictionary using a
`foreach` loop.

```csharp
foreach (var wordCount in wordCounts)
{
    Console.WriteLine($"{wordCount.Key} : {wordCount.Value}");
}
```

This is fine. It works, and we can easily see what is doing. But the naming of
things is not immediately clear, because it has more to do with the data
structure we're using than the problem we're trying to solve. First, we have to
have a name for each key-value pair in the dictionary. Secondly, the actual word
and its count are accessed using the Key and Value properties, the names of
which mean nothing in the business context. We could assign these values to well
named local variables, but that's just adding more code that obscures what we're
actually trying to do.

We can improve this slightly by enumerating the keys in the dictionary, rather
than the key-value pairs.

```csharp
foreach (var word in wordCounts.Keys)
{
    Console.WriteLine($"{word} : {wordCounts[word]}");
}
```

This is better, but there's still some ceremony in there. What happens if we
define a deconstructor for the key-value pair? I'm glad you asked.

```csharp
public static void Deconstruct<TKey, TValue>(
    this KeyValuePair<TKey, TValue> kvp, TKey key, TValue value)
{
    key = kvp.Key;
    value = kvp.Value;
}

foreach (var (word, count) in wordCounts)
{
    Console.WriteLine($"{word} : {count}");
}
```

Look at that! A clear enumeration with well-named variables that clearly
communicates the business concern with very little ceremony.

"But," you may say, "look at the variable names in the Decontruct method.
They're meaningless." To which I would reply, "no they're not." That's a generic
helper method that operates on any `KeyValuePair<,>`, with names that are perfectly
meaningful in that context. And really, what else are you going to call them?

### But what about performance?

I could quote Donald Knuth about the evils of premature optimization, but I
won't. Suffice to say, there's nothing to worry about. I ran some tests timing
100 enumerations of a dictionary with 100k entries using each of the three
techniques above. Each test run took 3-5 seconds to complete.

Enumerating the dictionary itself, as in the first example above, is the fastest
option. However, the third example, where we deconstruct the KeyValuePair, only
has about a 1% performance hit.  Given the improvement in readability, I'd say
that's worth it.

The worst option, performance wise, was the second example above where we
enumerate the keys and then index into the dictionary in the loop. That had a
30-50% performance hit over enumerating the dictionary itself.

### Conclusion

The new Deconstructor support in C#7.0 not only aids in handling ValueTuples,
but can also be used to pull apart others types such as arrays and dictionaries
with less code and greater readability.
