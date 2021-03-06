---
title: CSAPP-Chapter 1-Introduction
date: 2018-12-18 20:04:54
categories:
- 阅读分享
tags:
- computer system
comments: true
---

## Information is Bits in Context

bytes sequence

* text file
* binary file

## Programs are Translated by Other Programs into Different Forms

![compilation system](/images/CSAPP_1.3.png)

* preprocessor phase: modify line begin with #, marco substitute, conditional compile, file insert
* compilation phase: translate text file to assembly language
* assembly phase: translate assembly language to machine-language instructions
* linking phase: merging symbols from different object file, result is executable object file

## It Pays to Understand How Compilation System Work

reasons for why we need to understand how compilation system work?

* optimizing program performance: how C statement translate into assembly language, such as switch vs if-else
* understanding link-time errors: errors which happen during linking time, like can't resolve a reference
* avoiding security holes: buffer overflow bugs

## Processors Read and Interpret Instructions Stored In Memory

![hardware organization of a system](/images/CSAPP_1.4.png)

the system contains:

* bus
* i/o device
* main memory: DRAM
* processor: PC, Regsiter, ALU
  * load
  * store
  * update
  * i/o read
  * i/o write
  * jump

runing the hello program:

![reading the hello command from keyboard](/images/CSAPP_1.5.png)

![loading the executable from disk into main memory](/images/CSAPP_1.6.png)

![writing the output string from memory to the display](/images/CSAPP_1.7.png)

## Caches Matter

system spends a lot time moving data from one place to another

storage devices: larger vs smaller, slow vs fast, cheap vs expensive

use smaller faster storage device called cache to serve as temporary staging areas for data

L1/L2 caches use SRAM

![caches in system](/images/CSAPP_1.8.png)

## Storage Devices Form a Hierarchy

storage at one level serves as cache for storage at the next lower level

![storage hierarchy](/images/CSAPP_1.9.png)

## The Operating System Manages the Hardware

![abstraction provided by an operating system](/images/CSAPP_1.11.png)

* process
  context switch

* thread

* virtual memory
  * progress code and data
  * heap: malloc, free
  * shared libraries code and data
  * stack: implement function calls
  * kernel virtual memory: 1/4 of the address space

![linux process virtual address space](/images/CSAPP_1.13.png)

* file

## System Communicate With Other Systems Using Networks

![a network is another I/O device](/images/CSAPP_1.14.png)