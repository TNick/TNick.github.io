---
layout: post
title:  "Dynamic arrays in c"
date:   2014-03-05 13:00:00
categories: survey
---

This page attempts to gather information about various data structures used for 
storing dynamic content in c.


Simple dynamic arrays
---------------------

A simple implementation [here](https://gist.github.com/TNick/9364630). It usses
`realloc()` to change the size of the array, that may become slow. Also note
[this StackOverflow answer](http://stackoverflow.com/questions/19146037/efficiency-of-growing-a-dynamic-array-by-a-fixed-constant-each-time)
about the strategy to use when resizing. Since elements are continous in
memory may be fast when accessing content sequentially.

obstacks
--------

An obstack is a pool of memory containing a stack of objects. You can create any 
number of separate obstacks, and then allocate objects in specified obstacks. 
Within each obstack, the last object allocated must always be the first one freed, 
but distinct obstacks are independent of each other.

- [Wikipedia entry](http://en.wikipedia.org/wiki/Obstack)
- [socoder tutorial](http://socoder.net/?article=29407)
- [GNU page](http://gcc.gnu.org/onlinedocs/libiberty/Obstacks.html)
- [GNU guide](http://www.delorie.com/gnu/docs/glibc/libc_43.html)
- [C++ implementation](https://github.com/cleeus/obstack)
- [www.chemie.fu-berlin.de](http://www.chemie.fu-berlin.de/chemnet/use/info/libc/libc_3.html#SEC33)

Judy arrays
-----------

- [Sourceforge repo](http://sourceforge.net/projects/judy/) has latest version 1.0.5
released in May 2009
- [Home site at judy.sourceforge.net](http://judy.sourceforge.net/).
- `libjudy-dev` in Ubuntu
- [Wikipedia entry](http://en.wikipedia.org/wiki/Judy_array)
- [2003 testing by Sean Barrett](http://www.nothings.org/computer/judy/)
- [an implementation in 1250 lines](https://code.google.com/p/judyarray/), last update
ine June 2013; there is some work on [github by JanX2](https://github.com/JanX2/judy-arrays/wiki)
- [compression-code](https://code.google.com/p/compression-code/) 
Public Domain implementation of Adaptive Huffman Coding in C;
Implement Vitters modification to algorithm FGK in C code.
- [Hashtables vs Judy Arrays, Round 1 (2008)](http://rusty.ozlabs.org/?p=153)


Repositories on Github
- https://github.com/alanpearson/judy
- https://github.com/electromatter/judy-svn
- https://github.com/darconeous/judy
- https://github.com/dump247/judy
- https://github.com/tony2001/judy build system cleaned
- https://github.com/bpowers/judyarrays Markdown
- https://github.com/qiq/judy-fk This is a modified JudySL implementation, so that keys are not NUL-terminated C
strings, but fixed-length sequence of bytes. Except this, it is the same as
original JudySL implementation.
- https://github.com/felix-lang/judy


hat-trie
--------

[Project page](https://code.google.com/p/hat-trie/); written by Karl Malbrain.

A HAT tree consists of a cascaded series of root radix director nodes for 
leading key bytes, which then point to either another radix node, or a hash 
bucket, or a string array which contains the rest of the bytes in the keys

- [github.com/dcjones/hat-trie](https://github.com/dcjones/hat-trie)
This a ANSI C99 implementation of the HAT-trie data structure of
 Askitis and Sinha, an extremely efficient (space and time) modern variant of tries.
The version implemented here maps arrays of bytes to words (i.e., unsigned longs),
which can be used to store counts, pointers, etc, or not used at all if you
simply want to maintain a set of unique strings.


high-concurrency-btree
----------------------

Implement Lehman and Yao's high concurrency B-Tree in C language with 
Jaluta's balanced B-link tree operations including a locking 
enhancement for increased concurrency.

- [former google code repo](https://code.google.com/p/high-concurrency-btree/)
- [current github repo](https://github.com/malbrain/Btree-source-code)


Other things
------------

discovered while researching this:

- [Arena memory allocator in C](https://code.google.com/p/arena-memory-allocation/) - 
A bit map is used to track allocated memory in storage arenas;
- [byte-oriented-aes](https://code.google.com/p/byte-oriented-aes/) 
Implementation of a byte-oriented approach to the Advanced Encryption Standard 
in standard C code. By careful tracking of the input and output state array 
indicies, the MixColumns, SubBytes, and ShiftRows steps are combined into 
a single round operation;
- [regular-expression-evaluator](https://code.google.com/p/regular-expression-evaluator/)
A hybrid regular expression evaluator for xsd:pattern combines memoization with 
features from the DFA and NFA approaches to provide excellent run-time characteristics
with time O(mn) and space O(mn) where m is the pattern size and n is the source length.
- [secure-remote-password](https://code.google.com/p/secure-remote-password/)
A small footprint implementation of the Secure Remote Password protocol in C
- [kernel-doc](https://www.kernel.org/doc/Documentation/kernel-doc-nano-HOWTO.txt)
linux kernel documentation style
- C Code Archive network [website](http://ccodearchive.net/) and [repo](https://github.com/rustyrussell/ccan)



