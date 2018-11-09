---
layout:  post
title:   Meson for Mesa
date:    2018-11-09
summary: A short description of the current state of the build systems in mesa, the status of the meson build, why I chose meson, and where we're going in the future.
categories:
 - meson
 - mesa
---

# Where We've Been

## A Very Short History of Building Mesa

Mesa is a very old project by free software standards, it was first released in
1993, only a couple of years after Linux was. The first releases didn't use any
build system, just Make files. Then it got autoconf, and after many years a
full automake build system. Then Gallium came about (which is itself a long and
controversial subject), which needed to run on Windows, and they added scons
support for building on windows. For various reasons autotools stuck around for
pretty much every other platform (Haiku used scons), and the scons build system
could basically only build one software rasterizer on OSes other than Linux.
Then Google happened, and Android took off, and people wanted to use mesa on
Android, so we got yet another build system: Android make. This is a *very*
abbreviated version of events, but it's good enough to get us up to the point
that I really want to talk about.

## How We Got Meson

At this point mesa had 3 different build systems, all of them terrible in their
own ways, all of them good in their own ways. Autotools needs little
explanation, it's great because it's the de facto standard on FOSS OSes,
everyone knows at least a bit about it and they certainly have it installed;
it's terrible because it's written in m4, and requires writing make files
(which are awful for portability). Scons was great because it worked on
Windows, but terrible because it was slow, mesa's implementation doesn't look
much like other scons build systems, it uses lots of python-2.x-isms, and it
can't build most of the drivers. And Android.mk, well, it's all terrible, but
it's the price of Android.

At the time CMake was the build system everyone seemed to be reluctantly moving
toward. It offered a ninja backend and visual studio backends, it's open source,
it's pretty common, and we were using it for other projects (namely our piglit
test suite). Like many other Open Source developers who had to deal with
Windows and Unix-like OSes in a single project I'd accepted that CMake was
worth the pain to avoid having two build systems.

It was dealing with piglit that led me to change my mind. Apart from the things
most people recognize as short comings of CMake (which I wont go over), and
enterprise CMake (bat/sh script calls cmake, cmake calls bat/sh script, bat/sh
script calls cmake, ...), it allows developers to be clever. Build systems are
a bad place to be clever. Macros and functions often don't make the code
clearer, and the emphasis on proxy targets doesn't help either.

Someone mentioned meson. So I went and looked. The meson developers saw the
same problems with systems like CMake I did. The syntax was nice, and as a
python developer myself I saw a system that I could submit patches for instead
of trying to work around. It was cross platform. Meson also tried to have sane
defaults on *all* OSes (CMake has bad pkg-config integration and getting things
installed into the correct directories in Linux and the BSDs is a bunch of
boilerplate).

I'll add here as an aside, the IGT developers started looking at meson around
the same time I did, but we never really talked about it at the time. As far as
I remember I started playing with meson before gnome did as well, I think
GStreamer was the only major project at the time using meson, though I wasn't
aware of that until after I started.

So I started writing meson code, I started with mesa, but found it was just too
big and moved too fast for me to figure out meson (and really understand
autotools). Instead of I ported libdrm, then worked on mesa-utils to try to get
windows stuff working. Eventually I started porting mesa itself.

## All Along the Way

When I first started GStreamer was the only (maybe there were others) other
large project using meson, and it had a developer doing a lot of work on meson
itself. I also started working on meson because there were a lot of things that
we were doing that other projects weren't at the time. Among them: Extensive
code gen, mixing C and C++ liberally, using LLVM, making extensive use of
linking entire static archives together, array options, and support for a bunch
of OSes that meson hadn't really supported before (Haiku, the BSDs, Solaris).

LLVM has proved the biggest headache of the bunch. I could write an entire blog
post on why LLVM is an awful dependency, but it basically comes down to not
having pkg-config files, and a llvm-config's tendency to lie to you.

# Where we Are

Today, mesa has good meson support for all OSes except windows. There are a
couple of outstanding issues I'm working on: selecting a specific llvm-config,
support for the Intel C/C++ Compiler. And windows support is still a work in
progress, but it's coming along.

We also still have 3 other build systems, autotools, scons, and android.mk.
Haiku has migrated to meson from scons to meson and at least Archlinux and
Gentoo have started using meson by default instead of autotools.

# Where we are Going

The plan as of right now is to drop autotools during the 19.0 cycle (after the
18.3 release) presumably. We need to get llvm-config override support in meson
for that to happen, but I feel like we're pretty close to having that shored
up. ICC support in meson is in okay shape right now on Linux, though I have
additional patches to get it into better shape.

Windows work is ongoing, but I'm hopeful that it can be in shape to drop scons
in 2019 as well. Hopefully next year will be the year we delete scons as well.
Then we just need to get the android thing figured out.

{% include cc-by-sa-4_0.html %}
