---
layout: post
title:  "Pylearn2 basics"
date:   2015-03-30 19:49:00
categories: pylearn2
---

Overview
--------

[Pylearn2][pylearn2]  is a machine learning library.
Most of its functionality is built on
top of [Theano][th]. This means you can write [Pylearn2][pylearn2] plugins (new models,
algorithms, etc) using mathematical expressions, and [Theano][th] will optimize and
stabilize those expressions for you, and compile them to a backend of your
choice (CPU or GPU).

Official git repository is at [https://github.com/lisa-lab/pylearn2][pyl2git]

    git clone https://github.com/lisa-lab/pylearn2.git

and my clone with things not (yet) merged into main repo can be cloned with:

    git clone https://github.com/TNick/pylearn2.git

The library is research-oriented, so you mind find it dificult to get
it to work with your own dataset or to get it to work at all.

As of this writing the
[development mailing list](https://groups.google.com/group/pylearn-dev)
if very low trafic while the
[user mailing list](https://groups.google.com/group/pylearn-users) is active.



[pylearn2]: http://deeplearning.net/software/pylearn2/ "Pylearn2 homepage"
[pyl2git]: https://github.com/lisa-lab/pylearn2 "Pylearn2 GitHub repository"
[th]: http://deeplearning.net/software/theano/ "Theano homepage"

