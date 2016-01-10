---
layout: post
title: "Continuous integration testing of MPI projects with travis"
date: 2016-1-10
---

In hind sight I should not have been surprised: testing MPI
projects with travis is super easy!  I've created a
[minimal template](https://github.com/d-meiser/mpi-travis) that
illustrates the process.


## Installation of MPICH

Obviously, in order to test MPI code, we need an MPI library.
[OpenMPI](http://www.open-mpi.org) or [MPICH](https://mpich.org)
are the obvious choices.  Here we'll be using MPICH.

We could use the packages that come with the Linux distribution
on the travis instances but for a number of reasons I prefer
building MPICH from source.  For one thing it gives us complete
control over the MPICH configuration.  This way we can more
easily and more accurately replicate our local development
environment on the test instances.  To build MPICH we use the
following script:

{% highlight bash %}
if [ -f mpich/lib/libmpich.so ]; then
  echo "libmpich.so found -- nothing to build."
else
  echo "Downloading mpich source."
  wget http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
  tar xfz mpich-3.2.tar.gz
  rm mpich-3.2.tar.gz
  echo "configuring and building mpich."
  cd mpich-3.2
  ./configure \
          --prefix=`pwd`/../mpich \
          --enable-static=false \
          --enable-alloca=true \
          --disable-long-double \
          --enable-threads=single \
          --enable-fortran=no \
          --enable-fast=all \
          --enable-g=none \
          --enable-timing=none
  make -j4
  make install
  cd -
  rm -rf mpich-3.2
fi
{% endhighlight %}

The script first checks if we already have `libmpich.so`.  If so,
we are done and the rest of the script is skipped.  This
makes the script idem-potent:  calling the script a second time
yields the same state of the mpich installation.

If we don't find `libmpich.so` we have to do the actual work
involved in obtaining MPICH.  We download the source tarball,
extract it, configure, build, install, and clean up.  This takes
about 4 minutes on the travis test instances.


## Building and running the tests

To run the tests we use the following `.travis.yml`
configuration:
{% highlight bash %}
sudo: false
language: c
cache:
  directories:
  - mpich
before_install:
  - sh ./get_mpich.sh
script:
  - make test
  - ./mpich/bin/mpirun -np 4 ./test
{% endhighlight %}
The first line disables `sudo` for the test run.  This is
important because it allows us to use the container
infrastructure on travis.  The containerized instances are faster
than the regular ones and, more importantly, have access to
caching of build artifacts.  We cache the `mpich` directory which
is where we install MPICH with the `get_mpich.sh` script from
above.  Once the MPICH installation is cached the `get_mpich.sh`
finishes nearly instantaneously on subsequent test runs.  For
this template, an entire test run finishes in less than ten
seconds.

The actual test script is then super simple.  We simple build the
`test` executable and run it with the desired number of processes
using MPICH's `mpirun`.  Of course you need to tell the compiler
where to find the MPI headers and library.  In our case we do
this very simply using a handwritten gnu `Makefile`:
{% highlight make %}
CFLAGS+=-I./mpich/include
LDFLAGS+=-L./mpich/lib -Wl,-rpath,./mpich/lib
LDLIBS+=-lmpi
{% endhighlight %}

For completeness, here is the source of the `test.c` program:
``` c
#include <mpi.h>
#include <assert.h>
#include <stdio.h>

int main(int argn, char **argv)
{
	int status;
	int myRank;
	status = MPI_Init(&argn, &argv);
	assert(status == MPI_SUCCESS);

	status = MPI_Comm_rank(MPI_COMM_WORLD, &myRank);
	assert(status == MPI_SUCCESS);
	printf("Hello from rank %d\n", myRank);

	status = MPI_Finalize();
	assert(status == MPI_SUCCESS);

	return 0;
}
```

Of course this approach can be refined in many ways.  In a real
project you'll want to interface with your configuration system
(e.g. cmake) and you're tests will have to be hooked into your
test harness.  You could also imagine testing on several
different MPICH versions and with OpenMPI.  You can do that
easily using the travis test matrix, e.g. using environment
variables to control which MPI is built and used.
