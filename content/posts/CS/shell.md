---
title: "Shell操作指南"
date: 2022-11-01T19:43:16+08:00 
lastmod: 2022-11-01T19:43:16+08:00 
draft: false

keywords: ["bash"]
description: ""
tags: ["bash"]
categories: ["CS"]
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
    - `-l *.md`: 得到特定格式的文件
        ```bash
        missing % ls *.txt
        hello.txt	hello2.txt
        ```

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
- `{}`:  expand a common substring in a series of commands. 相同字符可以缩写，例如 `touch foo{,1,2,10}` 创建四个文档 - `foo`、`foo1`、`foo2`、`foo10`
    ```bash
    # This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
    touch {foo,bar}/{a..h}
    ```
- `<()`: execute and place the output in a temporary file and subsitute the <() with that file's name
    ```bash
    touch foo/x bar/y
    # Show differences between files in foo and bar
    diff <(ls foo) <(ls bar)
    # Outputs
    # < x
    # ---
    # > y
    ```

## 定义和输出变量
没有变量的时候单引号双引号都可以，有变量的话只有双引号起作用。
```bash
~ % foo=bar
~ % echo $foo
bar
~ % echo 'Value is $foo'
Value is $foo
~ % echo "Value is $foo"
Value is bar
```

## Write things in a file
```bash
# 运行文件
~ % source mcd.sh
~ % mcd test 
```
- `!!` - Entire last command 复制上一行指令
- `$0` - Name of the script
- `$1` to `$9` - Arguments to the script. `$1` is the first argument and so on
- `$@` - All the arguments
- `$#` - Number of arguments
- `$$` - Process identification number (PID) for the current script
- `$_` - Last argument from the last command
- `$?` - Return code of the previous command
```bash
~ % true
~ % echo $?
0
~ % false 
~ % echo $?
1
```

```bash
~ % foo=$(pwd)
~ % echo $foo
/Users/missing
```

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

# Others
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

# Finding tools
## Finding files
`.` 表示在当前文件夹下搜索，`-name` 后面为想要查找的文档名称
```bash
# Find all directories named src
~ % find . -name src -type d
~ % find . -name '*.md' 
./archetypes/default.md
./content/draft/promise.md
```
`locate` 用于在一个 `database` 里面查找文档，效率更高。`updatedb` 用于更新 `database`。

## Finding commands

`Ctrl + R` 检索你的历史指令

```bash
~ % history
 1043  man ffmpeg
 1044  tldr convert
 1045  tldr ffmpeg
 1046  find . -name src -type d
 1047  ls
 1048  cd public
 1049  ls
 1050  cd ..
 1051  cd snowcabin
 1052  find . -name src -type d
 1053  find . -name src -type md
 1054  find . -name '*.tmp'
 1055  find . -name '*.md'
 1056  locate shell
 1057  locate missing-semester
 1058  update db
 # only print commands that contains the substring 'locate'
~ % history 1 | grep locate
 1056  locate shell
 1057  locate missing-semester
```