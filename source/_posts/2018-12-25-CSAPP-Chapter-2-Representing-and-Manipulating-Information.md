---
title: CSAPP-Chapter 2-Representing and Manipulating Information
date: 2018-12-25 14:08:34
categories:
- CSAPP
tags:
- computer system
comments: true
---

three most important encoding of numbers:

* unsigned Integers
* signed Integers
* float-point number

we need to focus on:

* the ranges of values that can be represented
  * overflow
  * roundoff
* the properties of different arithmetic operations
  * integer arithmetic satisfy commutative and associative
  * float-point arithmetic is commutative, but not associative

## Information Storage

virtual memory -> larger bytes array -> bytes -> block of eight bits
virtual address space -> each memory bytes has a unique number

in C, pointer's value is the virtual address of the object's first bytes
compiler use the pointer's type to generate machine code to access object's value

### Hexadecimal Notation

binary decimal and hexadecimal

write bit pattern as base-16 or hexadecimal number

### Words

word size of a computer indicates the noraml size of integer and pointer data
virtual address is encoded by a word, the word size is the maximum size of the virtual address space
n word size -> [0, 2^n] virtual address space

### Data Sizes

different data type may have different size depend on the machine and compiler

### Addressing and Byte Ordering

object that span multiple bytes, we must establish two conventions:

* what will be the address of the object: smallest address of the bytes used
* how will we order the bytes in memory: big endian and little endian

![big endian vs little endian](/images/CSAPP_2.1.png)

when byte ordering becomes a issue:

* binary data is communicated over a network between different machines
* programs are written that circumvent the normal types system

### Representing Strins

### Representing Code

### Boolean Algebras and Rings

### Bit-Level Operations in C

### Logical Operations in C

### Shift Operations in C

## Integer Representations

## Integer Arithmetic

## Floating Point