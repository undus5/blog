+++
title       = 'CSAPP Post Reading'
subtitle    = ''
date        = '2025-12-29'
lastmod     = '2026-01-20'
tags        = []
showSummary = true
showTOC     = true
weight      = 1000
draft       = false
hidden      = false
+++

Computer system is no longer mystery to me.

<!--more-->

Computer Systems: A Programmer's Perspective, aka CSAPP, you will always see its
name when you search for book recommendations about programming or computer science,
so I bought it, 8 years ago, and things weren't going well, barely finished the
first two chapters. During the reading, there was always such a voice in my head,
like "Why am I reading these? They seem interesting, but hard and time-consuming,
I think I'm not gonna have chance to use these knowledge, maybe invest the time
on learning other cool stuff?", then finally I lost patient, couldn't read anymore.

That's how things started, and I felt frustrated in the following years, util
now I finished reading the book, I realized no need to feel in that way back then,
it just wasn't the right time for me to read the book, because I didn't truly
understand the value of those low level things at the moment. After years
of working, tinkering with operating system and softwares, there are so many
questions accumulated in my head not answered, and I can sense I need some
fundamental knowledge. The reading experience this time is a totally different
story, it feels like a breeze, an enjoyable journey, full of "Aha!" moments and
smiles.

The most important thing I get from the book is that computer system is being
demystified to me. It also cured my frustration and anxiety for facing that
blackbox. It gives me superman's x-ray vision, let me look inside the
system, learn how things work under the hood. Turns out there's no magic in
computer system, just a bunch of zeros and ones, representing different things
via some conventions and mechanisms designed by the pioneers. Since these
zeros and ones are contiguous, whether in ram or disk, or from network interface,
you need to split them into meaningful segments in some way, that's the reason
there're always header and maybe footer in the data structures like network
packets and even in memory spaces, which tell you where to begin, end and how to
decode the data.

When I learned what a ".so" library is, and also the overall principle of linking,
it hit me with a "Aha!".
Back in the time when I was tinkering with Linux, I often got the error
like "missing xxx.so library", and sometimes I needed to manually download and
copy the library files into /usr/lib/ or just specify the environment variable
LD_LIBRARY_PATH ahead of the running command. I knew the library is some
essential dependency, but I didn't know what it really is. Another one is about
the dynamic linker "ld-linux.so", which I already used a couple of times for
converting my VPS into Arch Linux. After deleting the old system, I need to run
the following command to initiate the new system with the help of ld-linux.so:

```
(root)# /root.x86_64/usr/lib/ld-linux-x86-64.so.2 \
    --library-path /root.x86_64/usr/lib \
    /root.x86_64/usr/bin/chroot /root.x86_64 pacstrap /mnt \
    base linux mkinitcpio grub openssh
```

When I learned about virutal memory page, it reminded me some
good old times, more than a decade ago I think, I was still a teenager at the time,
there was a list of tricks from the internet for improving Microsoft Windows
performance, one of them is about tweaking the virtual memory page size,
virtual memory page, what a mystery name to me at that very moment.

Everytime a "Aha!" moment came to me, it made me smile, it's just, you know,
that joy of acquiring knowledge.

The next book should be The Algorithm Design Manual according to
[teachyourselfcs.com](https://teachyourselfcs.com/),
but I am more interest in operating system, so I will try
[OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/) first,
wish me luck, to be continued.

