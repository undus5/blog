+++
title       = 'OSTEP Post Reading'
date        = '2026-04-23'
lastmod     = '2026-04-23'
tags        = []
showSummary = true
showTOC     = true
weight      = 1000
draft       = false
hidden      = false
+++

I can imagine what happens behind the scenes of every click.

<!--more-->

Operating Systems: Three Easy Pieces, or
[OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/),
in fact I've heard about it before
[teachyourselfcs.com](https://teachyourselfcs.com), and tried to read it once
since I was tinkering with Linux and wanted to dig deeper. The book
suggests readers to read CSAPP first at the very beginning in the introduction
chapter, I just ignored and kept reading, then I realized I could barely follow,
hehe ... So don't be arrogant, take some advices, and be patient,
Rome wasn't built in one day.

OSTEP is like an extended version of the last part of CSAPP, but from a
different perspective, I would call it a system designer's perspective.
It's also a pragmatic version, since the examples in the book are based on
real systems.

The most important thing I got from the reading is it helped me consolidating
this common framework in my head to understand computer system, which is,
there's always a giant byte array, you create some data structure to manage it,
with some header blocks following with data blocks or so, and there's always
some cache layers sit between some components for the sake of performance or
bandwidth, according to spatial or temporal locality.

It also reveiled how things work under the hood about something I've already
used before or something I've heard from tech news, such as shell
redirection, segmentation fault, thread-safe, copy-on-write, defragmentation,
TLC/QLC in SSD etc.

There's a 4th piece beyond the "three easy pieces" which is about security,
it's not written by the main authors. I skipped it because it feels a little
painful to read compared to the other three pieces, maybe retry on another day.

A little tip here: don't skip the appendix part, or you will miss a lot.
There's a chapter about virtual machine in the appendix part, this topic is
one of my top motivations of learning operating system, since I've already
tinkered with QEMU a lot. The laboratory tutorial about how to compile C project
is very practical and useful, you should read it even if you don't want to write
C program, I wish I could read it earlier, since I was always wondering how it
works every time I manually compiling some packages but didn't know how to get
started back to the early days. The laboratory projects are also interesting,
but I decide to read other books first.

Errata: Chapter 40 File System Implementation, page 13,
"one to write the bitmap (to reﬂect its new state to disk)" should be
"write the inode bitmap".

The next book is The Algorithm Design Manual, to be continued.

