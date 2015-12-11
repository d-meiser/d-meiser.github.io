---
layout: post
title: "How much of a difference does optimization of gcc make"
date: 2015-12-10
---

A couple of days ago I did a comparison of compilation times between
several builds of gcc.  The different builds were (all from gcc trunk,
revision ???):

- default build of gcc:
```
../gcc/configure \
  --prefix=/usr/gcc-trunk \
  --enable-languages=c,c++,fortran \
  --disable-bootstrap \
  --disable-werror
make -j4
make install
```

- default gcc with gold linker
```
../gcc/configure \
  --prefix=/usr/gcc-trunk \
  --enable-languages=c,c++,fortran \
  --disable-werror \
  --disable-bootstrap \
  --enable-gold
make -j4
make install
```

- profile guided optimized + gold linker
```
../gcc/configure \
  --prefix=/usr/gcc-trunk \
  --enable-languages=c,c++,fortran \
  --disable-werror
  --enable-gold
make -j4
make install
```

I used the differnt gccs to compile one of our Tech-X codes.  The code
base consists of about 300k lines of code (as measured with David A.
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


