---
title: A cheat sheet for … jQuery ?
tags:
- Reference
- Cheat Sheet
- jQuery
last_modified_at: '2014-03-04T04:21:23.198+11:00'
---
And now for something completely different ... a man with three buttocks!

Okay, so maybe not THAT different.

It's not C# or .Net, though. It is, however, something that many ASP.Net
developers will run across - jQuery.

Long story short - I've created a new jQuery cheat sheet in two versions
([A4][A4] and [Letter][Letter]). If you want the long story, read on.  If you
don't then just grab your copy and let me know if you find it helpful.
<!--more-->

![JQuery Cheat Sheet Thumbnail](/assets/images/jquery-cheat-sheet-thumb.png "JQuery Cheat Sheet Thumbnail")

12 months ago I'd never heard of jQuery, and then I started seeing bits and
pieces about it in posts on [Stack Overflow](https://stackoverflow.com). Then
when I started working on the ASP.Net MVC application at work, the one I've
mentioned previously, I found myself looking around for a good UI framework and,
lo and behold, there was [jQuery: The Write Less, Do More, JavaScript
Library](https://jquery.com).

For those of you who don't know, jQuery is a Javascript framework, written in
Javascript, that runs in the browser and makes in insanely easy to manipulate
the HTML DOM, and create all sorts of cool UI effects, Ajax interactions, and
heaps more.

As I write this, jQuery version 1.6 is in beta, and it's big. There are
somewhere in the vicinity of 500 property and method signatures, which begs the
question, "how do I find out what I can do with this thing?".

Well, there's the excellent on-line [jQuery API](https://api.jquery.com)
documentation, which is also available in a [dead-tree
version](https://www.packtpub.com/jquery-1-4-reference-guide/book) (albeit for
version 1.4), but it's not free, and is a bit more than I was looking for.

Of course, what every competent developer really needs is a quick reference, or
Cheat Sheet, to quickly get an idea of what operations are available, or to just
get a reminder of the arguments for a specific method call.  A quick look on
Google turns up quite a number of cheat sheets of various kinds.  The newest one
I found is the excellent [jQuery Visual Cheat
Sheet](https://www.scribd.com/doc/55305532/jQuery-1-6-Visual-Cheat-Sheet)
designed by Antonio Lupetti. Antonio's done a great job with this sheet, which
he appears to have first created for jQuery 1.3, and has updated for each
version of jQuery since.

As good as the Visual Cheat Sheet is, it's not quite what I was looking for.
For starters it's four pages long, and I was hoping for something I could print
on a single sheet of paper.  I was also hoping for something that had some
indication of the jQuery version in which the various methods and properties
were introduced.

At the same time I was looking for cheat sheets, I discovered that the jQuery
API docs are available as a [raw XML dump](https://api.jquery.com/api). So I
grabbed a copy and, inspired by [G. Scott Olson's jQuery 1.2 Cheat
Sheet](https://www.gscottolson.com/jquery/jQuery1.2.cheatsheet.v1.0.pdf), I set
to writing an XSL stylesheet to turn it into something readable. My goal was to
create a simple cheat sheet that would suit my needs, and that I could quickly
update for future jQuery versions. The XSL I came up with produces a chunk of
HTML, which I then copied into MS Word and tweaked the formatting.

The result is a two-page cheat sheet that can be printed double-sided on a
single sheet of paper, and lists every method, property, selector and template
tag signature in the XML documentation.  For each signature it also lists the
return-type and the jQuery version in which it first appeared.  Assuming no
major structural changes, I can produce a cheat sheet from a new version of the
XML in about 10 minutes.

Now I'm happy, and I want to share my happiness with you.  I'm making the cheat
sheet available to anyone who'd like to use it.  I've created two versions -
one for [A4 paper][A4] and one for [Letter][Letter]. Both have exactly the same
information.  Grab the one that suits your printer. And please, let me know if
you find it useful, or you notice any glaring omissions or errors.  Any
suggestions for improvements would also be welcome.

I would like to, at some stage, build on my XSL to create a jQuery eBook
containing a fuller reference with examples from the XML documentation, but I
have a few other things on my plate before I get to that.  It would also be
really cool to look at Microsoft's XML document formats so the stylesheet could
produce a native Word document that needs minimal tweaking.  There's always
something else to learn but, hey, that's what makes life interesting.

[A4]: /downloads/jQuery-1.6-Cheat-Sheet-A4.pdf
[Letter]: /downloads/jQuery-1.6-Cheat-Sheet-Letter.pdf
