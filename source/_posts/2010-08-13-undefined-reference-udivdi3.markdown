---
layout: post
title: "undefined reference to __udivdi3"
date: 2010-08-13 23:03:56 +0000
comments: true
published: true
categories: [linux, kernel]
---

During the last month I have been playing with real-time scheduling in the Linux kernel. I’ve downloaded the SCHED_DEADLINE implementation from their git repository and did some changes in some files in order to understand better how real-time task scheduling can be handled in Linux using this particular scheduling policy.
<!-- more -->
I used a 64-bit machine with Ubuntu and Eclipse CDT version to implement my modifications.

**You may wonder why do I use Eclipse?**  
The answer to that is simple. Eclipse has great code indexing and cross-reference features, which are very helpful in understanding the kernel's inner details.

Moving back to the subject of this post, in my 64-bit machine I’m able to compile a modified version of the kernel without any kind of problems but I’m not able to run the compiled version. After some discussion with an expert in the subject, he told me that probably I wasn’t compiling the correct modules that would allow me to run the OS without any problems. Thus, I decided to re-compile the same modified version in an old 32-bit machine which I wasn’t using anymore, to see if I was able to, at least, run the modified version of the kernel.

The processors I'm using are the following:

64-bit machine architecture: "model name    : AMD Turion(tm) X2 Dual-Core Mobile RM-74"
32-bit machine architecture: "model name    : Intel(R) M processor 1.60GHz"

I confess that initially I thought that the compilation process would be smooth, but I got surprised when the code didn’t compile and gcc threw an **Undefined reference to __udivdi3** error.

After some googling, I discovered that the error was thrown in a line where **a division involving u64 data types was being done**. I’ve found some good pointers to the problem, namely:

http://kerneltrap.org/mailarchive/linux-kernel-newbies/2009/2/5/4902104  
https://bugs.launchpad.net/ubuntu/+source/linux-meta/+bug/237528

So, according to the above posts, it looks like the problem is caused by a compatibility issue between the kernel sources and gcc v4.3. Well, that made sense to consider as I was using that version in the 32-bit machine. However, after an update of the gcc version to v4.4.3, I still got the same error in the 32-bit machine and, surprisingly, with the same version in my 64-bit machine, the kernel compiles smoothly. So, definitely, this was not a compiler's problem but, most probably, an error related to the machine’s architecture and how 64-bit division is performed.

After some googling (again), I’ve found this Stack Overflow thread that somehow fundaments my suspicions: https://stackoverflow.com/questions/1063585/udivdi3-undefined-how-to-find-the-code-that-uses-it

Indeed, it is an architecture's related issue and how division is being handled. After modifying the kernel's code to perform a 64-bit division in a 32-bit architecture, using the `uint32_t do_div(uint64_t dividend, uint32_t divisor)` function, the code compiled like a charm.

And now let’s run the kernel!