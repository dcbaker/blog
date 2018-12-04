---
layout:  post
title:   Why you don't want user defined functions and macros in your build system
date:    2019-12-04
summary: Nothing good comes of it
categories:
 - meson
---

# Why User Defined Functions and Macros are Bad

This is hopefully going to be a short post about why meson insists that you fix
bugs in meson itself, and not in your local scripts like many other build
systems do.

Lets start by talking about the two build systems people usually compare meson
to: autotools and cmake. Both of these languages have a scripting language,
autotools has the m4 macro language, and cmake has the cmake language.

[m4](https://en.wikipedia.org/wiki/M4_(computer_language)) is an old macro
processing language, it's difficult to debug and understand, not unless you do
a lot of autoconf you probably don't know anything about it. If you've seen an
autotools project, you've probably seen a folder called m4 with a bunch of
ax\_\*.m4 files in it. These are macro definitions that are used in the
configure.ac file. These files are copied from other projects into your m4
folder, and used during the build.

For CMake there's usually a folder called cmake, that has a bunch of *.cmake
files in it. Same idea, different language.

So why does this not work?

Mostly for two reasons:
 - version proliferation
 - missed code sharing

## Version Proliferation

Imagine that we have a macro called DoFoo, and it's in a file called
dofoo.file. Imagine that this file is part of a project called A. Now two other
projects, B and C, also need this script, so they copy the file. B copies
version 1, then A updates to version 2 which C copies. Now project B makes some
different changes, and project D copies that, then they make some more changes
and project E copies that. Meanwhile C makes changes to version 2, and project
F copies that. Now there are 6 copies of dofoo.file, each different, and each
broken in different ways.

The second problem is that each of these copies tends to only work on the
configuration that the developers of that project use. If they are working on a
Windows project, it works for Windows; if they work on Debian, it works for
Debian, etc.

This is what I'm calling version proliferation; there are dozens of versions,
each of them different, and each of them broken in different ways.

## Missing Code Sharing

We can as programmers all agree that sharing and reusing code is (generally)
better than not sharing and reusing code. This is why we have libraries. While
some heavily make use of vendoring to keep their own altered copies of
libraries in their source tree, most of us realize this is a bad practice, we
don't want a copy of zlib or openssl in our source tree, we want to keep
getting fixes from upstream. Shared scripts have the same problem, when the
code is in the core build system everyone gets to use and enjoy it without the
burden of maintaining it directly.

We can take mesa as an example. Mesa has 156 lines of macros for dealing with
llvm-config in configure.ac. Some of this works around different brokenness in
different versions of LLVM, some of it helps set up the right arguments to pass
to llvm-config. Every project that uses LLVM needs the equivalent of those 156
lines, and there are bugs for some configurations of LLVM in that logic. Meson
has 198 lines of code for dealing with llvm-config in core meson. That means
anyone who wants to use LLVM as a dependency can simply say:

```meson
dep_llvm = dependency('llvm')
```

and things just work.

{% include cc-by-sa-4_0.html %}
