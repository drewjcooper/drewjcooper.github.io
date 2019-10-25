---
title: Fun with initialisers
date: '2018-06-16T22:38:00.000+10:00'
tags:
- C#
modified_time: '2018-06-16T22:38:44.833+10:00'
---
C# has had object and collection initialisers since C# 3.0, but all
the docs and blogs I've ever seen only cover the basics. We've all seen
examples like these:\

    var address = new Address
    {
        Line1 = "1 Microsoft Way",
        City = "Redmond",
        State = "WA",
        Zip = 98075
    };

    var languages = new List<string>
    {
        "C#",
        "Java", 
        "Python"
    };

But have you ever stopped to think what's going on under the hood and
considered the implications? I want to go there, but first let's look at
dictionaries.\

### Dictionary Initialisation

We've always been able to initialise a dictionary with a collection
initialiser like this:\

    var months = new Dictionary<int, string>
    {
        { 1, "January" },
        { 2, "February" },
        { 3, "March" },
        // and the rest, you get the idea
    };

In C# 6.0 a new syntax was introduced, and in pretty much everything
I've ever read about it it's described as just that, a new way of
writing the dictionary initialisations we'd all come to know and love, a
different twist on Collection Initialisers. But the change in dictionary
initialisation was actually a side-effect of adding indexer support to
Object Initialisers. More on that later.\

### Collection Initialisers

Collection Initialisers work on any type that implements IEnumerable and
has an `Add(...)` method (or, since C# 6.0, extension method). Look at
the list of languages initialisation above. The compiler converts this
to the equivalent of:\

    var _c = new List<string>();
    _c.Add("C#");
    _c.Add("Java");
    _c.Add("Python");
    var languages = _c;

The months dictionary initialisation works because of a similar
transformation:\

    var _d = new Dictionary<int, string>();
    _d.Add(1, "January");
    _d.Add(2, "February");
    // and the rest
    var months = _d;

Note that this is basically then same as the list initialisation. The
only difference is the number of parameters taken by the Add method. If
you're anything like me, your next thought is, "can this go beyond 2
parameters?"\
Before I get to the answer, lets have another look at the requirements
for collection initialisers. The two examples above use collection types
that are included in the BCL, but that's not a requirement. The only
requirement is that the type implements IEnumerable, and that a suitable
Add method is available. So, my next thought after the number of
parameters was whether or not collection initialiser can handle multiple
Add methods. The answer is pretty cool, but I'm not sure it's useful for
much in real world code.\
It turns out that when the compiler is transforming the initialiser it
looks for an appropriate Add method for each element of the initialiser.
This means that you can actually have elements with differing numbers of
items in one initialiser, as long as the Add methods are there to
support them. Have a look at the following class:\

    class Foo1 : IEnumerable
    {
        public void Add(long a) => Console.WriteLine("Add with one parameter");
        public void Add(string s1, string s2) => Console.WriteLine("Add with 2 strings");
        public void Add(bool f, double d) => Console.WriteLine("Add with 2 different params");
        public void Add(int x, char y, string z) => Console.WriteLine("Add with 3 parameters");


        // We need this to implement IEnumerable, but not for this example
        public IEnumerator GetEnumerator() => throw new NotImplementedException();
    }

Note, that there's nothing vaguely collection-like about this type, but
it is a valid target for a collection initialiser. Here's an example,
and the output it produces:\

    new Foo1
    {
        { "foo", "bar"},
        { 42 },
        { 3, '9', "27" },
        { true, 1.0 }
    };

Output:\

    Add with 2 strings
    Add with one parameter
    Add with 3 parameters
    Add with 2 different params

So now that we've abused collection initialisers, lets turn our
attention elsewhere.\

### Object Initialisers

Before C# 6.0, object initialisers worked by setting fields or
properties of the type being initialised. The compiler translates the
Address initialisation at the top of this post into the equivalent of
the following:\

    var _a  new Address();
    _a.Line1 = "1 Microsoft Way";
    _a.City = "Redmond";
    _a.State = "WA";
    _a.Zip = 98075;
    var address = _a;

With C# 6.0 came support for setting indexers as well, and this is
where the new dictionary initialisation syntax comes from. But
initialising dictionaries is not the only place it can be used. Any type
with indexers can use this style of initialisation, and it can be mixed
with setting properties and fields. Consider this type:\

    class Foo2
    {
        public string bar;
        public double Baz { set => Console.WriteLine("Property setter"); }
        public char this[int i] { set => Console.WriteLine("Indexer with 1 index"); }
        public string this[char c, int i] { set => Console.WriteLine("Indexer with 2 indices"); }
    }

Here we have a field, a property, and two indexers with different number
and types of indices. The following initialiser:\

    new Foo2
    {
        bar = "bar1",
        [3] = '9',
        Baz = Math.PI,
        ['C', 4] = "Middle C"
    };

produces this output:\

    Indexer with 1 index
    Property setter
    Indexer with 2 indices

Again, not sure how useful this is in real life or maintainable code,
but it's interesting nonetheless.\

### And back to Dictionaries

I'm going to wrap this post up with an interesting difference in
behaviour when initialising dictionaries with the two different syntaxes
available. Consider the following two initialisations:\

    var d1 = new Dictionary<int, string>
    {
        { 1, "one" },
        { 1, "uno" }
    };

    var d2 = new Dictionary<int, string>
    {
        [1] = "one",
        [1] = "uno"
    }

On the face of it they look equivalent, but the behaviour is very
different. The first attempts to call the `Dictionary.Add` method twice
with the same key, resulting in an exception being thrown on the second
call. The second initialiser just sets the dictionary indexer twice,
resulting in a single entry with a key of `1` and a value of `"uno"`.\
I hope you\'ve enjoyed my exploration (and abuse) of initialisers in
C#. Till next time, "Happy Coding".\
\
