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
* [练习](https://github.com/MillyTang/missing-semester/blob/main/first.sh)

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
foo=bar
echo "$foo"
bar
echo '$foo'
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
# source 可执行shell文件
source testShellFunc.sh
# cd 是在 mcd()内部，针对的是子shell, 不影响外部，所以不能直接执行 ./testShellFunc.sh
```

### 短路操作符

> `||, &&, ;`, 同一行命令可以用`;`间隔，`true`的返回码是`0`,`false`的返回码是`1`

```bash
# 和js的短路操作符结果相似，这里唯一特别的是「;」，始终能运行短路后面的语句
true ; echo 'I can run!'
# 输出：I can run!
false ; echo 'I can run again!'
# 输出：I can run again!
```

> linux中： `| and ||, & and &&, &> and >` 的区别：
> `&` 表示任务在后台执行，如要在后台运行
> `|` 表示管道，上一条命令的输出，作为下一条命令参数(输入)
> `>`符号是指：将正常信息重定向; `&>`可以将错误信息或者普通信息都重定向输出

### 命令替换：以变量的形式获取一个命令的输出

> 执行`$(CMD)`,`CMD`的执行结果会替换`$(CMD)`的位置；
> 冷门应用-进程替换:`<(CMD)`, 执行结果将会输出到一个临时文件而不是通过`STDIN`传递
> 特殊的进程标识符： 0标准输入(stdin),1标准输出(stdout),2标准错误(stderr)

* `2>`重定向stderr；引申：`&>`重定向stderr, `>&2`，重定向`到`stderr
* `-ne` means `not equal to`, 查阅`bash 条件表达式`或`man test`(查test手册)
* 使用`shellcheck`工具检查shell script错误
* 详见[bash 条件表达式](http://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)
* [test手册](https://man7.org/linux/man-pages/man1/test.1.html)

```bash
#!/bin/bash
# testfunc.bash, 注意后缀是bash,而不是sh,因为`[[`判断符与sh不兼容
echo "Starting program at $(date)" # date会被替换成日期和时间
# `$$`进程ID，`$#`参数个数，$0当前命令
echo "Running program $0 with $# arguments with pid $$"
# `$@`所有参数
for file in $@; do
    grep foobar $file > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为 1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
# `source testfunc.bash`
# 输出： 
# Starting program at 2020年12月20日 星期日 14时52分03秒 CST
# Running program ./testfunc.bash with 0 arguments with pid 10741
# 加点参数执行 `source testfunc.bash test1 test2`, 同时在同级目录下生成了 test1 & test2文件并向文件中输入了 `# foobar`
Starting program at 2020年12月20日 星期日 14时54分51秒 CST
Running program ./testfunc.bash with 2 arguments with pid 10741
File test1 does not have any foobar, adding one
File test2 does not have any foobar, adding one
```

### shell的 通配（ globbing）: {}, 正则

```bash
convert image.{png,jpg}
# 展开为: 
convert image.png image.jpg
cp /path/to/project/{foo,bar}.sh /newpath
# 展开为: 
cp /path/to/project/foo.sh /path/to/project/bar.sh /newpath
mv *{.py,.sh} folder
# 会移动所有 *.py 和 *.sh 文件
# 显示foo和bar文件的不同
diff <(ls foo) <(ls bar)
```

### shebang

> `#!`语法使用脚本来指示解释器如何在`unix/linux`下执行
> 例如： `#!/bin/bash`;`#!/usr/bin/perl`;`#!/usr/bin/python`;`#!/usr/bin/python3`;`#!/usr/bin/env bash`
> 如果不指定解释器，通常会默认为`#!/bin/sh`,不过官方推荐使用`#!/bin/bash`,`sh和bash`在某些语法上有些不兼容，如条件判断`[[]]`

* 使用`env`来执行非`bash`脚本，`env`会利用`PATH`变量来定位脚本
* [参考](https://bash.cyberciti.biz/guide/Shebang#:~:text=It%20is%20called%20a%20shebang%20or%20a%20%22bang%22,using%20the%20interpreter%20specified%20on%20a%20first%20line.)

```bash
#!/usr/local/bin/python
# 使用python来解析该脚本
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

#### 查看命令

* `man -h`查看命令如何使用
* 使用`TLDR`，一个很好的替代品，`npm install -g tldr`全局安装

#### 查找文件

* `find`工具
* `fd`更快更友好
* `locate`只能通过文件名来定位，且每天都会更新

```bash
# 查找所有名称为src的 文件夹:d
find . -name src -type d
# 查找所有文件夹路径中包含test的python 文件:f
find . -path '**/test/**/*.py' -type f
# 查找前一天修改的所有文件
find . -mtime -1
# 查找所有大小在500k至10M的tar.gz文件
find . -size +500k -size -10M -name '*.tar.gz'
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {}.jpg \;
```

#### 查找代码

* `grep`对输入文本进行匹配的通用工具, `ack`, `ag`等
* `rg`，ripgrep速度快，符合直觉

```bash
# 查找所有使用了 requests 库的文件
rg -t py 'import requests'
# 查找所有没有写 shebang 的文件（包含隐藏文件）
rg -u --files-without-match "^#!"
# 查找所有的foo字符串，并打印其之后的5行
rg foo -A 5
# 打印匹配的统计信息（匹配的行和文件的数量）
rg --stats PATTERN
```

#### 查找shell命令

* `history`访问shell中输入的历史命令, `history | grep find` 会打印包含find子串的命令
* `Ctrl+R`命令历史记录进行回溯搜索，可以配合`fzf`使用，`fzf` 是一个通用对模糊查找工具
* 输入命令时，如果您在命令的开头加上一个空格，它就不会被加进shell记录中
* bash_history或 .zhistory 来手动地从历史记录中移除

#### 文件内导航

* `fasd`可以查找最常用和/或最近使用的文件和目录。基于 `frecency` 对文件和文件排序，也就是说它会同时针对频率（frequency ）和时效（ recency）进行排序
* 概览目录结构： `tree`, `broot`
* 文件管理器： `nnn`, `ranger`

#### 课后练习

* 第一题：`ls -AGulht`
* 第二题：[macro.bash](https://github.com/MillyTang/missing-semester/tree/main/second/macro.bash); [polo.bash](https://github.com/MillyTang/missing-semester/tree/main/second/polo.bash)
* 第三题：[check_run](https://github.com/MillyTang/missing-semester/tree/main/second/check_run.bash)
* 第四题：`find . -name '*.html' | xargs -0 zip -r output.zip` 注意文件名不要相同，否则zip会抛错停止
* 第五题： `find . ctime 10`

#### 其他练习&常用命令

ssh本地上传包文件

```bash
# 压缩dist文件夹
zip -r dist.zip dist
scp dist.zip username@hostname:/path/in/dir/
username@hostname'"s' password:
# 登录服务器查看文件是否上传成功
ssh username@hostname
username@hostname'"s' password:
cd /path/in/dir/
# 查看文件详细信息
ll
# 解压上传的dist.zip
unzip dist.zip
mv old-dir save-old-dir
mv dist old-dir
```
