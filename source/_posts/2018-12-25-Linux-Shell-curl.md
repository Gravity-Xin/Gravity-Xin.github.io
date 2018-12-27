---
title: Linux Shell curl
date: 2018-12-25 16:53:10
categories:
- Linux Shell
tags:
- curl
- http
comments: true
---

## Basics

curl support following protocals

* FILE
* FTP/FTPS
* GOPHER
* HTTP/HTTPS
* IMAP/IMAPS
* LDAP/LDAPS
* POP3/POP3S
* SCP/SFTP
* SMTP/SMTPS
* TELNET
* and etc

curl handle options and urls

* options
  * short options, like `-v`
  * long options, like `--verbose`
  * options with args, like `-d data`
  * options with args contains space, like `-d '{"key":"value"}'`
  * negative options, like `--no-verbose`

* urls
  * scheme
  * name and password
  * host name or address
  * port number
  * path
  * fragment

* url globbing
  * specify many urls
  * [] and {}

* use `curl --help` to list all options