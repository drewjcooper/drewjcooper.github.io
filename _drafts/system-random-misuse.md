# Intro

We've been interviewing for a new position at work recently, and we give all our
candidates a coding test to do, usually between the first and second interview.
The test involves design around caching of a key/value store in a multi-threaded
system. Part of the test requires generating a random set of keys for exercising
the cache. I've reviewed submissions from eight candidates so far and, to my
surprise, not a single one used `System.Random` correctly.

I was amazed at how little candidates for a mid to senior-level software
devlopement position knew about generating random numbers, much less
multi-threading. I don't know if it says more about the candidates, or the
design of `System.Random`, but I thought I'd write a few posts to highlight the
problems and show the right way (or ways) to do things.

# Threads, Seeds and Parameters


# Not thread safe
# Seeding
# Next params
# The right way
# Testing non-deterministic code deterministically
