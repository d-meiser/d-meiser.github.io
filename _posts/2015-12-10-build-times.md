---
layout: post
title: "How much of a difference does optimization of gcc make"
date: 2015-12-10
---

One of the perks of building gcc from source is that you get to customize it to
your needs.  One area of customization is the level of optimization with which
gcc itself is compiled.  This should have an impact on subsequent build times
with that compiler.  This can be very helpful when working on a large code base
or during testing (especially with continuous integration setups where the code
base needs to be built and rebuilt very frequently).  So how much of a
difference does an optimized ubild really make?

To find out I used different builds of gcc to compile several code bases.  Here
are the results.


## The contestants

For this comparison I built gcc's trunk (revision ) as follows:


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
  --prefix=/usr/gcc-trunk \
  --enable-languages=c,c++,fortran \
  --disable-werror \
  --disable-bootstrap \
  --disable-libquadmath --disable-libquadmath-support \
  --enable-gold
make -j4
make install
{% endhighlight %}


### profile guided optimized + gold linker

{% highlight bash %}
../gcc/configure \
  --prefix=/usr/gcc-trunk \
  --enable-languages=c,c++,fortran \
  --disable-werror
  --enable-gold
make -j4
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


