+++
title = 'Multithreading in C'
date = 2024-02-01T15:06:22-08:00
draft = true
+++

The purpose of this project is demonstrate how to create safe multithreaded applications in C, what can constitute a 'critical section', different techniques to address these race conditions and also understanding how these solutions can affect performance. Essentially, a critical section of code is simply where a shared data structure is accessed among different threads.

The project is split into 2 parts, which can be found on my Github [here](https://github.com/akhoshrozeh/multithreading-in-c/). 

Note: The meat of the analysis will be in the README's of my github.

### Part 1
We're going to analyze at 3 different types of synchronization mechanisms: mutex, spin-lock, and compare-and-swap. We'll be testing each mechanism with [yielding](https://en.wikipedia.org/wiki/Yield_(multithreading)) and without yielding as well. We'll also be looking at what happens when we don't address the race conditions for a shared variable in multithreading.