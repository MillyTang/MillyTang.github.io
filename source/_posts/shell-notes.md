---
title: Shell Notes
date: 2020-11-04 13:11:30
tags: shell,bash
---

## 第一章：概览

`shell:` 计算机提供给用户的一种文字接口，核心功能 - 允许用户执行程序，输入并获取某种半结构化的输出；
shell是一个编程环境，所以它具备变量、条件、循环和函数。
当你在 shell 中执行命令时，实际上是在执行一段 shell 可以解释执行的简短代码。
如果要求 shell 执行某个指令，但是该指令并不是 shell 所了解的编程关键字，那么它会去查询环境变量 $PATH，搜索由`:`分隔的一系列目录，基于名字搜索该执行程序

### 基本使用

#### shell的界面（bash@zsh）

`#` 开头是注释，提示信息，`hostname`是主机名，`~`指当前工作目录是`home`，`$`表示当前用户不是`root`用户，`Last login...`打开一个新的window就会出现该提示

```bash
Last login: Tue Nov  3 17:59:22 on ttys007
# hostname @ MacBook-Air in ~ [13:34:52]
$
```

#### 基础命令: 使用 `man [command]`来了解这些命令的详细信息

> 内置命令&变量: date, echo, which echo, $PATH, ls, cd, pwd, /(root path), ./(current path,relative), ../(parent path, relative), mv(用于重命名或移动文件), mkdir(新建文件夹), cp(复制文件)

```bash
$ # -l  use a long listing format
$ ls -l /home
total 0
drwxr-xr-x   3 user  staff   96 12  9  2016 beginning
  _3_           /|\
   |_____________|
# d -> directory
# 然后接下来的九个字符，每三个字符构成一组
# rwx -> 指(user)文件所有者的权限
# r-x -> 用户(staff)的权限，`-x`代表staff没有执行文件(x)的权限
# r-x -> 其他客人用户的权限
```

```bash
Last login: Fri Nov  6 18:13:54 on ttys007
# username @ MacBook-Air in ~ [13:34:52]
$ date
2020年11月 7日 星期六 16时09分15秒 CST
$ echo hello DaiYing
hello DaiYing
$ echo $PATH
/Users/hostname/.yarn/bin:/Users/hostname/.config/yarn/global/node_modules/.bin:/Users/hostname/.avn/bin:/Users/hostname/.nvm/versions/node/v8.9.4/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/hostname/.rvm/bin
$ which echo
echo: shell built-in command
```

#### 在程序间建立连接

> 输入流，输出流，重定向(重定向方向 <, >重定向方向, >>追加内容方向)
> | (pipes)管道操作符，连接左边的输入到右边的输出
> sudo(super user do)根用户

#### 练习题：Mac OSX系统

* 新建tmp/missing/semester文件，按行向semester输入内容

```bash
# username @ MacBook-Air in ~ [16:52:44]
$ mkdir tmp
$ cd tmp/
$ mkdir missing
$ cd missing/
$ touch semester
$ echo '#!/bin/sh' > semester
# 追加第二行内容
$ echo 'curl --head --silent https://missing.csail.mit.edu' >> semester
$ vim semester
# :q退出

```

## 第二章： shell工具和脚本

### 字符串区别

* `''`单引号可以转义
* `""`双引号不行

> shell访问变量: 前缀$+变量名；如：$foo

```bash
$ foo=bar
$ echo "$foo"
bar
$ echo '$foo'
$foo
```

### shell脚本控制流

> Here is an example of a func that creates a dir and  cd into it

* `$0`-script command
* `$1`~`$9`- script command的参数
* `$@` script command的所有参数
* `$#`- script command参数个数
* `$?`-上一个script command
* `$$`-当前scrippt command的PID(Process Identification number)
* `!!`-使用`sudo !!`执行上一个command(包含所有参数)
* `$_`-上一个命令的最后一个参数

```bash
# testShellFunc.sh
#!/bin/sh
mcd() {
  mkdir -p "$1"
  cd "$1"
}
# bash执行testShellFunc.sh文件
source testShellFunc.sh
# cd 是在 mcd()内部，针对的是子shell, 不影响外部，所以不能直接执行 ./testShellFunc.sh
```

### 短路操作符

> ||, &&, ;

```bash

```
