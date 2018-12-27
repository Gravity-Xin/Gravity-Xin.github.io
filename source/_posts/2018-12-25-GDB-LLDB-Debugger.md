---
title: GDB/LLDB Debugger
date: 2018-12-25 09:51:56
categories:
- Debugger
tags:
- gdb
- lldb
comments: true
---


## Debugger

a debugger is a program that simulates/runs another program and allows you to:

* pause and continue its execution
* set break points or conditions where the execution pauses so you can look at its state
* view and watch variable values
* step through the program line-by-line

## Use GDB

gdb is a GNU Debugger

* compile

to use gdb for debugging, you need use `-g` flag to compile
`-g` flag preserves the identifiers and symbols

```shell
gcc -Wall -g main.c
```

* start debugging

```shell
gdb a.out
# 增加程序的启动参数
gdb --args a.out arg1 arg2
```

* useful gdb commands
  * file: set executable file
  * refresh: refresh the display
  * set args PARAM: set args
  * run: run your program
  * backtrace full: see the crash stack
  * layout next: see your source code
  * list: list part of source code
  * break POINT: set a break point, use file_name:line_number, function_names and etc
  * condition: enter break point when condition is true
  * delete POINT: delete a break point
  * clear POINT: clear a break point
  * next: go to next step
  * continue: continue to next break point
  * step: step inside the function
  * print VARIABLE: print a variable's value
  * print *ARRAY@SIZE: print a array's value
  * watch VARIABLE: watch a variable for change
  * quit: quit debugger
  * help: show help of command

## Use LLDB