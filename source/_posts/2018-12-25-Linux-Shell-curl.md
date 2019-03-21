---
title: Linux Shell curl
date: 2018-12-25 16:53:10
categories:
- 编程技术
tags:
- curl
- http
comments: true
---

## 基本概念

根据提供的选项来处理URL

* 选项
  * 缩写形式, `-v`
  * 原始形式, `--verbose`
  * 带有参数值, `-d data`
  * 带有空格的参数值, `-d '{"key": "value"}'`
  * 使用 `curl --help` 来查看所有选项

* URL的组成
  * 协议
  * 用户名和密码
  * 主机名称或地址
  * 端口号码
  * 资源地址

## 使用方法

* verbose模式
    `-v` or `--verbose`: 获取更多细节信息
    `--trace [filename]` and `--trace-ascii [filename]`: 将细节信息保存到文件中
    `--trace-time`: 获取时间戳
    `-s` or `--silent`: 切换到silent模式
* 默认会复用之前的连接
* 下载相关
    `-o [filename]`: 指定保存的文件名称
    `> [filename]`: 指定保存的文件名称
    `-O`: 从URL获取保存的的文件名称
    `-J`or `--remote-header-name`: 从响应头中的参数中获取保存的文件名称
    `--compressed`: 要求服务器对响应结果进行压缩，然后在客户端进行解压
    `--limit-rate [speed]`: 设置平均下载速率
    `--max-filesize`: 设置下载文件大小的上线
    `--retry [number]`: 设置出错之后的重试次数
    `--continue-at [number]` and `--range [start-end]`: 断点续传
* 上传相关
  * HTTP
    `-d` or `--data`: POST请求参数
    `-F`: POST的multipart格式的参数
    `-T`: PUT的文件参数
  * FTP
    `-T`: 上传的文件参数
* 连接相关
    `--connect-timeout`: 建立连接的超时时间
    `--no-keepalive` and `--keepalive-time`: 默认使用keeplive
    `-m` or `--max-time`: 命令执行最大时间

## 使用Curl进行HTTP请求

![http with curl](/images/HTTP_With_Curl.png)