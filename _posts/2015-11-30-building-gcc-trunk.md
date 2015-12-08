---
layout: post
title: "Building gcc from source"
date: 2015-11-30
---

Tools for C++ have made incredible progress over the last few years.
With every release `gcc` and `clang` are becoming nicer in many ways
[^1]:

[^1]: The same could be said for MSVC, but I rarely program on windows
      so can't say much about that.  Also, as of this writing it's not
      possible to build MSVC from source.  Given the many moves
      Microsoft's C++ team has made towards opening up their eco system
      I wouldn't be shocked if that were to change in the near future.

- Better and nicer warning and error messages.
- New tools like address and undefined behavior sanitizers.
- Improved support for current and future C++ standards (C++11, C++14,
  and C++17).
- Improved support for advanced CPU instruction sets like AVX, AVX2, and
  AVX512.
- Improved auto-vectorization; support for new parallel
  programming specifications like OpenMP 4.x, OpenACC, and Cilk.
- Off-loading to compute accelerators; experimental features like
  compiling for NVIDIAs ptx instruction set.
- and so on.

With things changing this rapidly and in such exciting ways it's
sometimes nice to build the very latest compiler from source.  That's
especially true if the stock compiler that comes with your linux
distribution is dated (e.g. centos 6 which ships with gcc version 4.4).
In addition to the benefits just mentioned this also gives access to
experimental features that aren't in the release version of the compiler
yet.  As a bonus, you get to taylor various features of the compiler to
your needs.  For example, with gcc you get to chose the linker that it
uses which can make a fair bit of difference in linker speed.  Finally,
isn't it just great fun to be using the latest and greatest?

This post gives a quick walk through of how I go about building `gcc`
from source.  Turns out the many folks working on `gcc` have made it
pretty damn easy to build it from scratch.  So here's what you do.

First, you need to get the source.  For the trunk of `gcc` you use
{% highlight bash %}
svn co svn://gcc.gnu.org/svn/gcc/trunk gcc-trunk
{% endhighlight %}

To compile `gcc`, a few libraries are needed.  Getting the libraries is
super easy with the help of script that the `gcc` maintainers have
bundled with the source.  Much easier than getting and installing these
libraries by hand, making sure that they're the right version, and that
`gcc` is able to find them, etc.
{% highlight bash %}
cd gcc-trunk
./contrib/download_prerequisites
{% endhighlight %}
Running this script from the top level of the `gcc` source tree is
important in order for the configuration scripts to find the libraries
automatically.

Next, you need to configure the build of gcc.  There are a bunch of
environment variables that can mess up the build.  It's a good idea to
make sure that all of these are unset before doing the configuration.

{% highlight bash %}
unset C_INCLUDE_PATH CPLUS_INCLUDE_PATH CFLAGS CXXFLAGS
{% endhighlight %}

We configure the build out-of-source.  This way we don't pollute the
source tree with any build artifacts.  And we can clean up after
building simply by blowing away the build directory.
{% highlight bash %}
cd ../
mkdir gcc-trunk-build
cd gcc-trunk-build
{% endhighlight %}
`gcc`'s configuration uses autotools.  It's pretty impressive setup.
The `configure` script in the root directory is more than 16k lines long
at the time of this writing!! Gulp.

Usually I configure gcc as follows.
{% highlight bash %}
../gcc/configure \
    --prefix=/usr/gcc-trunk \
    --enable-languages=c,c++,fortran \
    --disable-libquadmath \
    --disable-libquadmath-support \
    --disable-werror \
    --disable-bootstrap \
    --enable-gold
{% endhighlight %}
This will build the C, C++, and fortran compilers.  The quadmath options
disable support for 128bit floating point numbers.  Without the
`--disable-werror` option gcc's compilation will fail if a warning is
encountered.  This happens occasionally, especially early in the gcc
release cycle.  So I turn `werror` off.  The `--disable-bootstrap`
option is also kind of interesting.  By default, gcc is built three
times.  You can find details of the process and reasons why this is done
on the
[gcc configuration page](https://gcc.gnu.org/install/configure.html).
With the `--disable-bootstrap` option it's built just once resulting in
a much quicker build process.  Finally, I usually use the `gold` linker
rather than the default `ld` linker because it tends to be faster for
C++ projects.

The configuration should finish rather quickly and after that you're all
good to go to build the compiler with

```
make -j4
```

For a non-bootstrapping build this should take no more than about 20
minutes.  Finally, to install use

```
make install
```

Depending on the install prefix you may need super user privileges for
this step.

To use the new compiler and to run the programs generated with it you'll
have to make sure that it's libraries can be found.  The easiest way to
do that is by adding `/usr/gcc-trunk/lib64` to the `LD_LIBRARY_PATH`
environment variable:

```
export LD_LIBRARY_PATH=/usr/gcc-trunk/lib64:$LD_LIBRARY_PATH
```

It's important to note that gcc's C++ standard template library,
`libstdc++` has undergone ABI incompatible changes in going from the 4.x
series to the 5.x series.  Although the whole story is more complicated
it's probably easiest to make sure that your project is built with a
consistent compiler and libraries.


## Sometimes things break in trunk

Something cool happened when I went through this recipe to make sure it
all works.  I got a compile error.  Cut and pasting the error message
into google lead me to this thread:

[https://patchwork.ozlabs.org/patch/551802/](https://patchwork.ozlabs.org/patch/551802/)

Apparently an issue with an incompatible version of `isl`.  The
bug had made its way into gcc's trunk only a couple of hours earlier.
Amazingly, google had already indexed the thread and lead me to the
discussion.  The next morning things were fixed again.


## Links and further reading

- [http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-pass2.html](http://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-pass2.html)
  Somebody else's recipe.
- [https://gcc.gnu.org/install/index.html](https://gcc.gnu.org/install/index.html)
  Detailed documentation of the configuration and build process.
- [https://gcc.gnu.org/wiki/InstallingGCC](https://gcc.gnu.org/wiki/InstallingGCC)
  An overview of the configuration and build process.
- [https://gcc.gnu.org/install/configure.html](https://gcc.gnu.org/install/configure.html)
  Documentation of the configure options.

