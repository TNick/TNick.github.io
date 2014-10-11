---
layout: post
title:  "Micro-modules"
date:   2014-05-24 12:00:00
categories: ideas
---

Overview
--------

This post attempts to organize ideas related to code re-usability.
It mostly targets pieces of code too small to form a library
but flexibility is added to create static or shared libraries.
We will refer to such a unit as a `pile`.

Each such `pile` consists of two repositories: 

- one that only contains the source code, copyright information, usage information, called `pile-source`
- one that contains everything else like test code, tools used to generate the source, etc. called `pile-helpers`

The word `pile` is to be replaced by the actual name.

The tools to be used are cmake and git. The parameters that change
the functionality are to be passed on using cmake, with
the source being kept unchanged. Any change to the source should
be passed upstream to improve the overall quality and usability.

CMake
-----

A `pile` uses CMake to generate content. The root directory of a 
`pile-source` contains a `CMakeLists.txt` file to allow including
the `pile` as a normal directory and it also contains a `cmake` directory
having a mandatory `pile.cmake` file.

`CMakeLists.txt` should only contain code specific for using the `pile`
as a normal library, either shared or static. All other functionality
is to be placed in `cmake/pile.cmake` so that the user has the option
to either `add_directory()` or to include the macro file.


Git
---

Git sub-modules are to be used extensivelly. Each `pile` is intended
to become a git submodule in the user's repository. To add a pile
we do  

    git submodule add git://github.com/chneukirchen/rack.git rack

To clone a project with submodules one has to 

    git clone git://github.com/schacon/myproject.git
    git submodule init
    git submodule update
    
    
Resources
---------

- [LXC on Ubuntu](http://srdandukic.blogspot.ro/2012/04/lxc-on-ubuntu.html)
- [Git Submodules](http://git-scm.com/book/en/Git-Tools-Submodules)