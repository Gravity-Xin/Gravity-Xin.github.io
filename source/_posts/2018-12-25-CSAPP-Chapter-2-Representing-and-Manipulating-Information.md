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

* unsigned integers
* signed integers
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

binary, decimal and hexadecimal

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

### Representing String

in C, string is an array of characters terminated by the null character

### Representing Code

different machine use different and incompatible instructions and encodings
binary code is seldom portable across different combinations of machine and operating sytem

### Boolean Algebras and Rings

boolean operations: ~ not, & and, | or, ^ exclusive or
extend the boolean operations to also operate on bit vectors

### Bit-Level Operations in C

C support bit-wise boolean operations, ~ & | ^
can be applied to any "integral" data type

### Logical Operations in C

C provide a set of logical operators, && || !

### Shift Operations in C

C provide a set of shift operations for shifting bit patterns, << >>

two forms of right shift:logical and arithmetic

## Integer Representations

unsigned for negative, zero and positive numbers
signed for nongative numbers

### Integral Data Types

C support a variety of integral data types, with an indication of unsigned

### Unsigned and Two's Complement Encodings

use two's complement form to represent signed numbers

### Conversions Between Signed and Unsigned

![signed to unsigned](/images/CSAPP_2.11.png)

![unsigned to signed](/images/CSAPP_2.12.png)

### Signed vs Unsigned in C

C support both signed(default) and unsigned arithmetic of integer data

C allow conversion between unsigned and signed, explicit and implicit conversion

### Expanding the Bit Representation of a Number

zero extension for unsigned number
sign extension for signed number

### Truncating Numbers

drop the high-order bits

### Advice on Signed vs Unsigned

conversion between signed and unsigned leads to some nointuitive behavior

* never use unsigned numbers

## Integer Arithmetic

### Unsigned Addition

may overflow, drop the higher order bit

![overflow](/images/CSAPP_2.15.png)

### Signed Addition

may negative or positive overflow, drop the higher order bit

![overflow](/images/CSAPP_2.17.png)

### Signed Negation

### Unsigned Multiplication

may overflow, drop the higher order bit

### Signed Multiplication

may overflow, drop the higher order bit

### Multiplying by Powers of Two

use shift operation, shift to the left, drop the higher order bit

### Dividing by Powers of Two

use shift operation, shift to the right

## Floating Point

### Fractional Binary Numbers

positional notation, like decimal

### IEEE Floating-Point Representation

V = (-1)^S × M × 2^E

S: 符号位
M: 有效数字，大于
E: 指数位

![32_bit](/images/CSAPP_32_Floating_Point.png)

![64_bit](/images/CSAPP_64_Floating_Point.png)