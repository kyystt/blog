---
title: Introduction to fuzzing
link: introduction-to-fuzzing
date: 2026-11-05 14:18:01
description: Notes from my first fuzzing experiments
tags:
  - Fuzzing
  - AFL++
  - Linux
categories:
  - Research
colophon:
  - handwritten
---

## Start
It all started when I watched this video

[![Finding The .webp Vulnerability in 8s (Fuzzing with AFL++)](https://markdown-videos-api.jorgenkh.no/youtube/PJLWlmp8CDM)](https://youtu.be/PJLWlmp8CDM)

Then i talked to a friend of mine ([Davi][Davi blog]) about it because he's been fuzzing things for a while (and 4 fun xD).

So i decided to try it myself, i guess thats the best way to learn

## Leap of Faith

I got a recomendation of a [github repo][fuzzing101], so i decided to give it a try.
The first real challenge here was setting up [AFL++][AFL++] and understanding its environment

<details open>

<summary>What is AFL++?</summary>

According to the [AFL++ docs][], AFL++ is a (better) fork from Google's AFL[^1].
It is coverage-guided, i.e., it tries to explore more of the program's behavior

Basically, AFL++ generates new input via initial _corpus_[^2], mutating it just like a genetic algorithm, and sending it back to the main application to see what happens

### Why is this useful?
You can think that the program is a giant maze, where each segment is like a room inside this maze.
The objective of the fuzzer is to explore all those rooms to see if theres any combination of possible inputs breaks the program

A simple random-input fuzzer generates inputs without using coverage feedback and hopes to reach the most ammount of rooms possible, it didn't save anything and had a few heuristics. A coverage-guided fuzzer (like AFL++) saves the input that reached a new room of the maze and start to mutate this input to see if he reaches any further

There are a lot of fuzzers out there and all have his own feature, you have to understand what do you want to do and analyze if this fuzzer makes sense
or if exists any other fuzzer that does the intended job better.

</details>

## Fuzzing101
On this repo, you'll learn by recreating CVEs[^3] actively (and trying to write a fix yourself).

You can just look at the CVE identifier and read it, find an input that triggers it by fuzzing and call it done. But, for me, to reproduce a CVE is trying to find it and investigate it as if no one had found it before. Of course you need to know what you're trying to reproduce, but that's the only information i guess you should use.

I wondered whether AFL++ had gotten faster since the exercises were written, ecause my run found a crash much sooner than the exercise’s estimate. After finding the crash, just hop on GDB and investigate it.

**I plan to write another post for solving fuzzing101 exercises**

### Harness
Sometimes (maybe most of the time) you'll try to fuzz a library or something like that, which means you need a program that calls its code.

On [libexif][], for example, you can use the binary `exif`, but you can create your own code that interacts with the lib, we call this code _harness_

## Libexif

On the second exercise of fuzzing101, the objective is to fuzz `libexif 0.6.14` and reproduce 2 CVEs, [CVE-2009-3895][] and [CVE-2012-2836][]. My 
first thought was to create a harness to work with this lib, my mind went blank and i forgot that the binary `exif` existed (and i did read the README from the libexif github repo).

I ended up trying both: my own harness and the exif binary. After a few minutes of fuzzing, I found some crashes and started investigating them in GDB. I’m still working through the root causes and haven’t yet confirmed whether they match the exercise’s CVEs.

It looks quite simple but it does teach you some important things, like reading real world open source library code and trying to understand how 
it works and dealing with ASan[^4] output with GDB crashes to find the root cause. Sometimes this can really be a pain, because not always the 
problem is where it crashed, maybe it is a wrong read size, pointer arithmetic with user inputs, improper casting, well, you've got it.

I got to learn about AFL++ itself, since it asks for you to use `afl-clang-lto`. Initially i was using `afl-clang-fast` to compile. AFL++ has this 
compiler wrapper that injects code instrumentation into the target program
> Remember when I said that AFL++ keeps inputs that increase coverage? this instrumentation is how AFL++ tracks coverage

Both are LLVM based

| `afl-clang-fast` | `afl-clang-lto` |
| ------------- | -------------- |
| instruments source file during compilation | instruments source file during linking |
| default PCGUARD instrumentation provides collision-free coverage | LTO[^5] instrumentation provides collision-free coverage |

> There are a lot of features that i haven't mentioned for both wrappers, you can check for your own in their [docs][AFL++ docs]

## Next steps
We can combine our current fuzzing session with `CMPLOG`[^6] and `COMPCOV`[^7] (help to check complex conditions and magic numbers) with automatic dictionary genearation, but these features are less useful if you don't understand what you're fuzzing and what you expect to achieve fuzzing the application

Finding a crash was only the beginning. Understanding the input, reading the library’s code, and figuring out why it failed took more work than starting the fuzzer.

I want to walk through the fuzzing101 exercises and explain the investigation in more detail, it means more posts soon.

## References

- [vanhauser's blog on fuzzing][vanhauser blog]
- [AFL++ features][AFL++ feats]
- [AFL++ best practices][]
- [Fuzzing101 Exercise 2](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%202)
- [LiveOverflow's video on libwebp fuzzing](https://youtu.be/PJLWlmp8CDM?si=66kWqzHKETaU4eLv)

[^1]: American Fuzzy Lop.
[^2]: A collection of test inputs used as starting points for fuzzing, also called seeds.
[^3]: Common Vulnerabilities and Exposures
[^4]: AddressSanitizer
[^5]: Link-time optimization
[^6]: Comparison logging
[^7]: Comparison coverage
<!---
Links
--->

[Davi blog]: https://davizin.com/
[AFL++]: https://aflplus.plus/
[fuzzing101]: https://github.com/antonio-morales/fuzzing101
[AFL++ docs]: https://aflplus.plus/docs/
[AFL++ feats]: https://github.com/AFLplusplus/AFLplusplus/blob/stable/docs/features.md/
[AFL++ best practices]: https://github.com/AFLplusplus/AFLplusplus/blob/stable/docs/best_practices.md/
[libexif]: https://github.com/libexif/libexif
[CVE-2009-3895]: https://www.cve.org/CVERecord?id=CVE-2009-3895/
[CVE-2012-2836]: https://www.cve.org/CVERecord?id=CVE-2012-2836/
[vanhauser blog]: https://srlabs.de/blog/advanced-fuzzing-unmasks-elusive-vulnerabilities/

