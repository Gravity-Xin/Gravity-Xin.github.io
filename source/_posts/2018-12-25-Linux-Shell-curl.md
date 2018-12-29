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

* use `curl --help` to list all options

## Use Curl

* verbose mode

    use `-v` or `--verbose` to get more information

    use `--trace [filename]` and `--trace-ascii [filename]` to store complete stream into file

    use `--trace-time` to store time

    use `-s` or `--silent` to switch curl to silent mode

* persistent connections

    curl will always try to keep connections alive and reuse the existing connections

* downloads

    use `-o [filename]` to save the download to a file

    use shell redirects `> [filename]` to save the download to a file

    use `-O` to download a file named by the file name in url

    use `-J`or `--remote-header-name` to get the file name from the response header

    use `--compressed` to ask server to provide compressed version of the data and perform automatic decompression on arrival

    use `--limit-rate [speed]` to setup the maximum average speed

    use `--max-filesize` to set the maximum bytes of filesize

    use `--retry [number]` to set retrying failed attempts

    use  `--continue-at [number]` and `--range [start-end]` to resume downloads

* upload
  * HTTP
    use `-d` or `--data` to post data

    use `-F` to post multipart form

    use `-T` to put file
  * FTP
    use `-T` to upload file

* connections

    use `--connect-timeout` to set the connect timeout

    use `--no-keepalive` and `--keepalive-time` to setup keepalive

* timeout

    use `-m` or `--max-time` to set the curl running time

## HTTP with Curl

GET is default, use `-d` or `-F` to POST, use `-I` to HEAD, use `-T` to PUT

![http with curl](/images/HTTP_With_Curl.png)