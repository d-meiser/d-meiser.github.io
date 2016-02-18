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
start from a clean slate.

So to figure out how to install packages in guile I picked the
zeromq bindings as a test piece.  The zeromq bindings where
originally developed by Andy Wingo.  The links at
[http://zeromq.org/bindings:guile-binding](http://zeromq.org/bindings:guile-binding)
are dead but you can find his version of the bindings at
[https://gitorious.org/guile-zmq/guile-zmq.git/](https://gitorious.org/guile-zmq/guile-zmq.git/).
Andy's zeromq bindings target the zeromq API version 2 or older.
However, my goal was to target newer versions of the zeromq API.
Surely others have run into this issue so I kept looking and
found a fork by Kjell Andreassen
[https://github.com/rashack](https://github.com/rashack) that
upgraded the bindings to the version 3 API.  My test repo is
based off of Kjell's code.

In this post I describe the steps needed to set up continuous
integration for `guile-zeromq-3`.  You can find the code in the
github repo
[https://github.com/d-meiser/guile-zeromq-3](https://github.com/d-meiser/guile-zeromq-3).
Hopefully this can serve as a more generally useful recipe for
other guile packages as well.


## Using the containerized infrastructure

When setting up travis configurations I usually try very hard to
avoid `sudo`.  This has several benefits.  First of all, it allows
me to use travis' containerized testing infrastructure.  With the
containerized infrastructure your test jobs start up faster
because it's more efficient to spin up a container than a whole
virtual machine instance.  And once your test jobs have started
you get an additional speed advantage because travis provides you
with more virtual cores when you're using the containerized
infrastructure.  There is yet another benefit: The
containerized infrastructure allows you to cache build artifacts.
With the traditional test instances you have to pay for caching.

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
build guile from source.  It's possible that there is a binary
package that could be used but I didn't research that option.

To build guile from source we need a number of prerequisites.  We
use travis' declarative syntax to specify the apt packages:

```
addons:
  apt:
    packages:
    - libgmp-dev
    - libunistring-dev
    - libgc-dev
    - texinfo
    - libzmq-dev
    - lcov
```

Strictly speaking, only the first four packages (`libgmp-dev`,
`libunistring-dev`, `libgc-dev`, `texinfo`) are needed to build
guile itself.  The zeromq library (`libzmq-dev`) is obviously
needed since we're trying to build the zeromq bindings.  And we
need `lcov` to measure the tests code coverage.

Once the dependencies are installed we're able to build guile.
For that I'm using the following script:

```bash
#!/bin/sh

if [ -f guile/bin/guile ]; then
  echo "guile found -- nothing to build."
else
  if [ -f guile-2.0.11/libguile.h ]; then
    echo "guile sources found -- don't need to download"
  else
    echo "Downloading guile source."
    wget ftp://ftp.gnu.org/gnu/guile/guile-2.0.11.tar.gz
    tar xf guile-2.0.11.tar.gz
    rm guile-2.0.11.tar.gz
  fi
  echo "configuring and building guile."
  cd guile-2.0.11
  ./configure \
          --prefix=`pwd`/../guile
  make -j4
  make install
  cd -
fi
```

This script first checks if guile has already been built.  If the
guile executable is found the script skips to the end.  If the
guile executable isn't found we download the source (if
necessary), configure, build, and install.  I cache the guile
installation and source directories:

```
cache:
  directories:
  - guile
  - guile-2.0.11
```

Building guile on the travis instances takes about 18 minutes,
well short of the one hour time limit for test jobs.  But after
guile has been built and cached subsequent test runs complete in
about a minute!


## Building the zermoq bindings

I configure, build, install, and test the zeromq bindings right
from the `.travis.yml` file:

```
script:
  - ./autogen.sh
  - CFLAGS="-fprofile-arcs -ftest-coverage" LDFLAGS="-lgcov" ./configure --prefix=`pwd`/guile
  - make
  - make install
  - guile -e main examples/hello-srv.scm &
  - guile -e main examples/hello-client.scm
```

Down the road it may be nice to wrap some of these steps into
separate scripts.  The first line generates the `configure`
script.  The `autogen.sh` in Kjell's repo didn't work for me.  So
I replaced it with a fresh copy from
[http://buildconf.brlcad.org/](http://buildconf.brlcad.org/).
Later I discovered that Andy has fixed this differently with
[this patch](https://gitorious.org/guile-zmq/guile-zmq.git/?p=guile-zmq:guile-zmq.git;a=commitdiff;h=31eec1691f80128e86e5ec1cd3b0d917301b4a6a).

I added a few compiler and linker flags to the configure line to
get test coverage data.  The only other thing that took me a
while was the prefix.  It wasn't clear to me whether this needed
to point to guile's site packages directory or the extensions
directory or what.  Turns out you just point it to the top level
of the guile installation.  I think it would be nice if the
`configure` script were written in such a way that the site
packages directory and extensions directories are determined
automatically based on what guile were configuring for.  That's
similar to how setuptools does things in python.  Using
`guile-config` it should be rather easy to implement this.
Unfortunately I don't know much about autotools ...

Once the bindings are built and installed we run the example
provided in the repo as a test that everything is working.


## Conclusion

I'm mostly happy with the setup of the test project.  I certainly
have a better idea of how to install packages with guile.  There
are just a couple of loose ends.


### Coverage data isn't working yet

The generation of coverage data isn't working yet.  When I run
the tests locally I see that `gcno` files are being generated,

```bash
$ find . | grep gcno
./guile-zmq/libguile_zmq_la-guile-zmq.gcno
./guile-zmq/.libs/libguile_zmq_la-guile-zmq.gcno

```

but there are no `gcda` files.  The `gcda` files are needed by
`lcov` to generate the coverage report.


### Error loading shared library

On one local machine I keep getting the following error message:

```bash
scheme@(guile-user)> ,use (zmq)
While executing meta-command:
ERROR: In procedure dynamic-link: file: "/scr/dmeiser/g2/lib/guile/2.0/extensions/libguile-zmq", message: "file not found"
```

But the shared library is there (with world readable and
executable permissions):

```bash
$ ls /scr/dmeiser/g2/lib/guile/2.0/extensions/
libguile-zmq.a  libguile-zmq.la  libguile-zmq.so
```

On my other laptop things work just fine.  Does anybody know how
to debug this?
