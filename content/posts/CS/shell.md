---
title: "Shell操作指南"
date: 2022-11-01T19:43:16+08:00 
lastmod: 2022-11-01T19:43:16+08:00 
draft: false

keywords: [""]
description: ""
tags: [""]
categories: [""]
author: ""

hidemeta: false
comments: true
# canonicalURL: "https://canonical.url/to/page"
# disableShare: false
# disableHLJS: false
# hideSummary: false
# searchHidden: false
# ShowReadingTime: true
# ShowBreadCrumbs: true
# ShowPostNavLinks: true
# ShowWordCount: true
# ShowRssButtonInSectionTermList: true
---

记录一些 shell 指令
<!--more-->

# 基础指令
- `date`： 日期
    ```bash
    missing % date 
    2022年11月 1日 星期二 19时45分27秒 CST
    ```

- `echo`: execute a command with arguments
    ```bash
    missing % echo hello\ world
    hello world

     missing % echo $PATH
    /Library/Frameworks/Python.framework/Versions/3.10/bin

    missing % which echo
    echo: shell built-in command
    ```

- `pwd`: get the path
    ```bash
    missing % pwd
    /tmp/missing
    ```

- `cd`: to a specific directory, `.` refers to the current directory, `..` refers to the parent directory
    ```bash
    missing /home % cd ..
    missing % pwd
    /
    
    missing % cd home
    missing /home % pwd
    /home
    ```
- `ls`: see what lives in current directory
    ```bash
    missing % ls
    Applications		Postman
    Aurora				Public
    Desktop				Zotero
    ...
    ```
    - `-h / --help`: get help
    - `-l`: use a long listing format

- `man`: have the manual page

# 操作文档

- `cp`: copy a file
    ```bash
    missing % cp markdown.md ../food.md
    ```
- `rm`: remove a file
    ```bash
    missing % rm ../food.md
    ```
- `mv`: rename or move a file
    ```bash
    missing % mv hello.md food.md
    ```
- `mkdir`: make a new directory
- `rmdir`: move a directory

# 连接项目
- `< file`: input from this file
- `> file`: output to this file
- `>>`: append file

```bash
missing % echo hello > hello.txt
missing % cat hello.txt
hello

missing % cat < hello.txt > hello2.txt
missing % cat hello2.txt
hello

missing % cat < hello.txt >> hello2.txt
missing % cat hello2.txt
hello
hello
```

- [`|`](https://en.wikipedia.org/wiki/Pipeline_(Unix)): pipeline, the output text of each process (stdout) is passed directly as input (stdin) to the next one
```bash
missing % ./semester.txt | grep -i last-modified 
last-modified: Fri, 28 Oct 2022 00:33:31 GMT
```
- `cat`: concatenate and print files

# 其他
- `chmod`: change file modes or Access Control Lists
    ```bash
    missing % ls -l
    total 8
    -rw-r--r--  wheel  61 11  1 20:13 semester.txt
    missing % chmod +rwx semester.txt
    missing % ls -l
    total 8
    -rwxr-xr-x  wheel  61 11  1 20:13 semester.txt
    ```
- `touch`: change file access and modification times
    ```bash
    missing % touch semester.txt
    ```
- `curl`: transfer a URL