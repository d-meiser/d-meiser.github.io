---
layout: post
title: "Comparison of build times with optimized gcc"
date: 2015-12-10
---
A couple of days ago I did a comparison of build times between several
builds of gcc.  The builds were:

- default build of gcc
- default gcc with gold linker
- link time optimized + gold

A word about the code base:
- loc
- libraries it links
- level of templatization

The results were

48m33s
58m36s
58m50s

Just relinking (`touch`ing one of the libraries)
35s
35s
52s


