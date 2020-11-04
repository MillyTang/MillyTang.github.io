---
title: Shell Notes
date: 2020-11-04 13:11:30
tags: shell,bash
---

# 第一章：概览

`shell:` 计算机提供给用户的一种文字接口，核心功能 - 允许用户执行程序，输入并获取某种半结构化的输出；
shell是一个编程环境，所以它具备变量、条件、循环和函数。
当你在 shell 中执行命令时，实际上是在执行一段 shell 可以解释执行的简短代码。
如果要求 shell 执行某个指令，但是该指令并不是 shell 所了解的编程关键字，那么它会去查询环境变量 $PATH，搜索由`:`分隔的一系列目录，基于名字搜索该执行程序

## 基本使用

### shell的界面（bash@zsh）

`#` 开头是注释，提示信息，`hostname`是主机名，`~`指当前工作目录是`home`，`$`表示当前用户不是`root`用户，`Last login...`打开一个新的window就会出现该提示

```bash
Last login: Tue Nov  3 17:59:22 on ttys007
# hostname @ MacBook-Air in ~ [13:34:52]
$
```

### 基础命令

> 内置命令&变量: date, echo, which echo, $PATH

```bash
Last login: Tue Nov  3 17:59:22 on ttys007
# username @ MacBook-Air in ~ [13:34:52]
$ date
2020年11月 4日 星期三 13时48分16秒 CST
$ echo hello DaiYing
hello DaiYing
$ echo $PATH
/Users/hostname/.yarn/bin:/Users/hostname/.config/yarn/global/node_modules/.bin:/Users/hostname/.avn/bin:/Users/hostname/.nvm/versions/node/v8.9.4/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/hostname/.rvm/bin
$ which echo
echo: shell built-in command
```