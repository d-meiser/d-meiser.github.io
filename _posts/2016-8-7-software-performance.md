---
layout: post
title: "Efficiency of software does matter"
date: 2016-8-7
---

"... Premature optimization is the root of all evil ..." - D. Knuth

  As a user, I really like it when a website is responsive.  As a user, I like
it, when the mail client on my phone has a fluid, low latency user interface.
As a user I like the nearly instantaneous download of the super useful utility
program that's merely 100KB in size.  As a business, I like it when I can run
the backend of my service on 10 instead of 100 servers, cutting my operating
expenses from $200k to $20k.

  Yes, efficiency of software does matter.  And not just in one of those nerdy
niche areas like gaming and high performance computing for scientific
simulations ... and virtual reality and augmented reality, high frequency/low
latency trading, gene sequencing, machine learning, machine vision, and big
data.

  Sure, as a creator of software I dislike having to worry about performance.
It forces me to work in a lower level language than I would if performance
wasn't a a concern.  I have to make concessions to how computers work when it
comes to choosing data structures and algorithms.  Encapsulationg of the
components of my software is not as great as it could be because I have to
exploit implementation details of one component in another.  And it's quite
possible that I'll have to write more code becuase I have to have custom code
for a bunch of important special cases.  What was that oft misquoted comment by
D. Knuth again?

  So we have two groups of people.  On the one hand we have users who are all
for performance.  On the other hand we have developers who don't want to deal
with the optimization needed to delver the desired performance.

  So it's a wash whether we optimize our software or not, right?

  Wrong.

  When judging the merits of software we must give much more weight to the
users.  Software with tons of developers and no users is worthless.  Successful
software has hugely more users than developers.  It's pretty much the
definition of successful software.  And the need of the many outweighs the
needs of the few.

  A different way to look at this is to note that the pain of optimizations
needed to achieve a certain level of efficiency can be quantified by the
additional development cost incurred by the optimizations.  The more successful
software is, the more the cost of software development is amortized over a
large user base.

  "But", you say, "users also want software with many features that doesn't
crash.  And they want it cheap."  True, goodness of software has several
distinct dimensions.  Feature set, stability, price, and efficiency are but
four examples.  Successful software strikes a balance between these dimensions.
And of course the relative importance of the dimensions depends on the
application.

  I'm not suggesting that every line of code should be as efficient as
theoretically possible.  That's the premature optimization Knuth is talking
about in his famous quote.  But in almost every piece of software some form of
efficiency (absolute performance, performance per watt, performance per dollar,
...) is an important feature.  Efficiency does matter ... at least for
successful software.

