---
layout: post
title:  "I've Lost My Memory"
date:   2026-07-16 12:09:00 -0400
---

With all the chaos caused by the global RAM shortage, I'll offer some insight into how it affects "normal" ML practitioners like myself.  But first, a quick primer on RAM and how it's used.

A modern computer (including phones, laptops, servers, etc) has four essential parts:

| CPU | a general-purpose calculator for processing data |
| GPU | a special-purpose calculator for very specific data tasks |
| RAM | short-term storage to hold data while the CPU or GPU are processing it | 
| SSD | long-term storage that persists when the device is off

LLMs, and what is commonly called "AI" depends very heavily on GPUs because they are required for the type of math (maxtrix algebra) that LLMs use to function.  Because GPUs can have 10,000 or more "cores" running in parallel, they need to use a special type of RAM which can feed data to them fast enough: GDDR.

But very many other ML tasks don't need GPUs at all -- often they don't use neural networks or matrix algebra.  They can run on CPUs with 10-30 "cores", which can use a slower (and cheaper) RAM: DDR.  This is the most common type of memory, and it's what I use for the ML tasks in my work.

The big difference between GPUs using GDDR and CPUs using DDR are speed and quantity.  For that, a quick overview of common RAM amounts is helpful:

| RAM | Speed | Maximum | Example
| ----------- | ----------- | ----------- |
| GDDR6 | 500-1500 Gbps | 80GB | Server GPU |
| DDR5 | 200-500 Gbps | 512GB | Desktop PC |
| DDR5 | 200-500 Gbps | 8GB | Smartphone |

The other important difference is RAM required for **training** versus RAM required for **inferencing**.  Typically, training a model (neural network or otherwise) takes ~100x more RAM than using it to make inferences (the "useful" part that most people are familiar with).  So an LLM model running on a smartphone with 8GB of memory would have required an AI cluster with, eg. 10 server GPUs and over 800GB of (expensive GDDR6) RAM to train it -- and this is a conservative estimate.

This train/inference asymmetry can actually be *more* pronounced for non-GPU built models.  One example is an algorithm I use, [MLLM](https://github.com/NatLibFi/Annif/wiki/Backend:-MLLM), which produces good results but is very memory-hungry.  Here is a graph of a training run on a vocabulary of ~10 million labels, with a training set of ~90 thousand documents, on a VPS with 2 cores and 256GB of RAM:

![VPS training run](/assets/image/training-ram.png)
*(DigitalOcean control panel)*

After several earlier attempts, the successful training run starts around 20:00 on the graph.  We can see the diagonal line rise from 20% to 100% RAM usage as it loads the training data into memory.  This part of the process took 4 hours and used all of the 256GB available.  Then it drops precipitously to about 55% RAM use for another 3 hours.  So memory use for ML is somewhat unpredictable and uneven throughout the process.

This training dataset of ~90 thousand documents is only part of a total dataset of ~5 million documents.  Extrapolating from this example, it will take ~11 days to train the entire dataset.  At $2/hr for the VPS, that's about **$600, 60kWh and 30kg CO2.**

What if we could do it faster?  It's possible to speed up some algorithms by breaking the workload into "jobs" and distributing them across multiple "cores"; eg. with 10 cores, 10x faster.  But there's a catch: each extra core requires extra memory, often double -- so with 10 cores, 10x more RAM.  Not only does this limitation reduce the training speed, but datasets which are too large to fit into the available RAM can't be trained at all!

Suddenly a physical limit of 512GB becomes very restrictive; in fact, the maximum for a DigitalOcean VPS is 384GB.  Apple recently discontinued their highest 256GB Mac and replaced it with a limit of 96GB.  This is a problem, but is has a (rather old) workaround: swap memory.

Modern operating systems have long had the ability to create "artifical RAM" by allocating portions of an SSD and essentially treating it as an extension of the computer's RAM.  The downside is that accessing swap memory is ~10x slower than "real" RAM.  MacOS makes use of this technique by using very fast SSDs and swapping heavily once RAM is about 85% used.

For the example above, let's say I want to use 8 CPU cores to make the training run faster.  I'd need 4 x 256GB = 1TB of RAM -- far more than the 384GB I can get from a VPS.  So I create a 768GB swap memory on the VPS and suddenly, I have 256GB + 768GB = 1TB of "RAM", with the caveat that it's about 65% slower.

But training the full dataset will now take ~3 days instead of 11.  At $3.5/hr for the VPS, that's about **$250, 20kWh and 12kg CO2.**