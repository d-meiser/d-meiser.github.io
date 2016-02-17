---
layout: post
title: "Setting up travis continuous integration testing for a guile project"
date: 2016-2-16
---

The mechanics of installing modules and libraries is often one of
the biggest hurdles I struggle with when learning a new
programming language.  For me, the fastest way to understand the
ins and outs of library installation and management is to set up
a continuous integration project using travis.  This way I'm
forced to spell out in detail the steps needed to install a
library.  And there is a clear definition of what it means for
the library to be installed successfully.  Frameworks like travis
make this whole process very well defined because you always
start from a clean slate.  There are non-deterministic
interactions with other packages or odd commands you happened to
run in your environment.

So to figure out how to install packages in guile I picked the
zeromq bindings as a test piece.  The zeromq bindings where
originally developed by Andy Wingo.  The links at ... are dead
but you can find his version of the bindings here.  When I
initially tried to build this library it turned out that it was
written for the pre version 3 API of zeromq.  Surely others have
run into this issue so I kept looking and found a fork by Kjell
Andreassen
[https://github.com/rashack](https://github.com/rashack) that
upgraded the bindings to the version 3 API.  My test repo is
based off of Kjell's code.

In this post I describe the steps needed to set up continuous
integration for that package.  Hopefully this can serve as a more
generally useful recipe for other guile packages as well.


## Using the containerized infrastructure

When setting up travis configurations I usually try very hard to
avoid sudo.  This has several benefits.  First of all, it allows
me to use travis' containerized testing infrastructure.  With the
containerized infrastructure your test jobs start up faster
because it's more efficient to spin up a container than a whole
virtual machine instance.  And once your test jobs have started
you get an additional speed advantage because travis provides you
with more virtual cores when you're using the containerized
infrastructure.  And you get an additional benefit: The
containerized infrastructure allows you to cache build artifacts.
A benefit you have to pay for when using the traditional test
workers.

There is an additional, purely pedagogical benefit.  I find that
I understand the build and installation process more thoroughly
if I don't spray all over my system files with `sudo this` and
`sudo that` and just hoping that things get installed in the
right default locations.  By installing outside of system
directories I'm forced to direct every component of the software
under test to a very specific location.


## Installing guile 2.0 on travis

The first thing needed to test a guile project is, rather
unimaginatively, guile itself.  The zeromq bindings require at
least version 2.0.  For the purposes of this project I chose to
build guile from source.

To build guile from source we need a number of prerequisites:

- libgmp-dev
- libunistring-dev
- libgc-dev
- texinfo

I use `apt-get` to install these using the system package manager.




