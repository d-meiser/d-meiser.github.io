---
layout: post
title: "Optimizing gcc build times"
date: 2015-12-10
---

One of the perks of building gcc from source is that you get to
customize it to your needs.  For example you get to choose the level of
optimization with which gcc itself is compiled.  So how much of a
difference does an optimized build make?


## The contestants

For this comparison I built gcc's trunk (revision 231276, last modified
on 12/4/2015) as follows [^1]:


### default build of gcc:

{% highlight bash %}
../gcc/configure \
  --prefix=/usr/gcc-trunk \
  --enable-languages=c,c++,fortran \
  --disable-bootstrap \
  --disable-libquadmath --disable-libquadmath-support \
  --disable-werror
make -j4
make install
{% endhighlight %}


### default gcc with gold linker

{% highlight bash %}
../gcc/configure \
  --prefix=/usr/gcc-trunk-gold \
  --enable-languages=c,c++,fortran \
  --disable-werror \
  --disable-bootstrap \
  --disable-libquadmath --disable-libquadmath-support \
  --enable-gold
make -j4
make install
{% endhighlight %}


### profile guided optimized + gold linker

I use the `profiledbootstrap` target to get a build of gcc with profile
guided optimization.  This target gathers profiling information during
one of the bootstrap stages and uses that in the final build stage that
produces the compiler executables.  This obviously means that we cannot
disable the bootstrapping process.  The following build takes quite a
bit of time (a few hours typically):
{% highlight bash %}
../gcc/configure \
  --prefix=/usr/gcc-trunk-pg-gold \
  --enable-languages=c,c++,fortran \
  --disable-werror \
  --disable-libquadmath --disable-libquadmath-support \
  --enable-gold
make -j4 profiledbootstrap
make install
{% endhighlight %}


## Results

The code base consists of about 300k lines of code (as measured with David A.
Wheeler's 'SLOCCount',
[http://www.dwheeler.com/sloccount/](http://www.dwheeler.com/sloccount/)).
It is moderately templated and links against a decent number of third
party libraries including boost, trilinos, hdf5, etc.

The results were

- default build with ld linker: 58m50s
- default build with gold linker: 58m36s
- profile guided optimization + gold linker: 48m33s

So profile guided optimization shaves off about 10 minutes, or nearly 20
percent.  That's more than I would have expected.  Pretty good given
that you get that speedup for free.

At first I thought the build time difference between the ld build and
the gold build might be noise.  To test that hypothesis I relinked the
executables in the project.  The link times were

- default build with ld linker: 52s
- default build with gold linker: 35s
- profile guided optimization + gold linker: 35s

So that's consistent with the timing difference we saw for the full
builds.  gold shaves off about 30 percent from the link times.


[^1]: Word on the street is that link time optimization (lto) also makes
      a difference.  I tried to create a build with lto and profile
      guided optimization but failed pretty badly.  Apparently lto and
      the profile guided optimization.  I kept running into compiler
      errors complaining about mismatching control flow between the
      measurement run and the second compilation run.  Eventually I
      managed to hack my way around these issues but the resulting
      compiler was about a factor of five slower than the results
      presented here.  Most likely user error.  Will have to try lto by
      itself to see if that leads to better results.
