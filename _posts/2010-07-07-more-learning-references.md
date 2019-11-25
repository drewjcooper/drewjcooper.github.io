---
title: More learning references
tags:
- Learning Process
last_modified_at: '2010-07-08T11:21:08.454+10:00'
---
As I've been working on learning C# and .NET in the last couple of
weeks, I've come across a number of useful references beyond the
tutorials and other things I list in [my second
post]({% post_url 2010-06-07-where-to-begin %}).
They range from books, to posters, to podcasts, and I'll briefly look at
them in this post.
<!--more-->

## Class Library references and posters

MSDN has a very complete reference of the [.NET Framework Class
Library](https://msdn.microsoft.com/en-us/library/ms229335.aspx), but,
given the shear number of namespaces and classes in the framework, it's
difficult to just browse and find something that looks like what you're
looking for.  What I want is a basic reference, preferably a PDF that I
can print, that lists the namespace hierarchy, along with the classes,
structs and other types in each namespace, and a brief description of
each.  In my hunt for such a reference I came across a couple of posters
illustrating portions of the .NET class library:

- The [.NET Framework 3.5 Namespace
  poster](https://blogs.msdn.com/b/brada/archive/2008/01/12/net-framework-3-5-namespace-poster-updated.aspx),
  as the name suggests, shows the commonly used namespaces and types
  in the 3.5 version of the .NET framework.
- The [.NET Framework 4 and
  Extensions](https://download.microsoft.com/download/E/6/A/E6A8A715-7695-493C-8CFA-8E0C23A4BE1D/098-115952-NETFX4-Poster.pdf)
  poster shows the namespaces and types that are new in 4.0.

There was another poster I found of the new stuff in version 4.0 which I
downloaded and printed, but now that I come to write about it I can't
seem to find it again to link to, and the file seems to have disappeared
from my disk.  Weird.

Anyway, these posters weren't exactly what I was looking for.  If anyone
knows of a reference such as I've described above, please let me know. 
Failing that I may have to put my growing C# skills to the test by
writing a small program to parse the HTML of the MSDN Library and
produce the reference myself.

## Coding style guidelines

Given that the programming I have done over the last 10 years or so has
been fairly ad-hoc, and in a range of languages, I felt a bit
out-of-touch with current thinking regarding coding style.  MSDN has
some guidelines, but they're mostly around naming conventions.  I wanted
to find something a bit more comprehensive.  I found a few blog posts
that talked about various aspects of coding style, but then I came
across Lance Hunt's [C# Coding Standards for
.NET](https://weblogs.asp.net/lhunt/pages/CSharp-Coding-Standards-document.aspx).
In Lance's words:

> There have been numerous attempts to document C# Coding Standards since the
> language was released, but most are either overly verbose, too restrictive, or
> try to cover every single scenario.  This is my attempt to start from scratch
> and write a new standards document that is concise, simple to read & use, and
> creates a pragmatic balance of rule enforcement.  Having said that, I'm sure
> that many of you will have just as many disagreements with this document as I
> have with others, but I am always up for a good debate.

The document is only 20 or so pages long and is a good, concise style
reference.  I'm not going to be following it religiously, but I will use it as a
guide as I write my code.

## Blogs, forums and podcasts

I've also looked around for resources I can use to keep up on developments in
the .NET universe, and also communities where questions can be answered, etc.  I
don't really have the time for reading too many blogs, but I do get time to
listen to podcasts, and to that end I found two that I'm now listening to:

- [Hanselminutes](https://hanselminutes.com) is a weekly audio talk show with
  noted web developer and technologist Scott Hanselman and hosted by Carl
  Franklin. Scott discusses utilities and tools, gives practical how-to advice,
  and discusses ASP.NET or Windows issues and workarounds.
- [.NET Rocks!](https://dotnetrocks.com) is a weekly talk show for anyone
  interested in programming on the Microsoft .NET platform. The shows range from
  introductory information to hardcore geekiness.

In my search for resources a few forum sites kept popping up in my Google
searches. The ones that really annoy me are the ones that make the questions
freely viewable, but then want to charge you a subscription to see the
answers. A great alternative is [Stack Overflow](https://stackoverflow.com/), a
free programming Q & A site -- lots of questions and quick-response answers on
just about any language/platform you can think of.

## Books

Last, but definitely not least, in this list of newly-discovered resources, is a
book that I found through listening to Hanselminutes - [.NET Book Zero by
Charles Petzold](http://charlespetzold.com/dotnet/index.html).  Yes, it's
\*that\* Charles Petzold, the one whose other book (one of many) taught me
Windows 3.1 programming back in Uni.  This book is a great introduction to C#
and .NET, and the best bit is ... it's free!

I think that will do for this post.  I'll put links to some of these
resources in the side bar.  Next post we'll get back to some code.  See
you then.
