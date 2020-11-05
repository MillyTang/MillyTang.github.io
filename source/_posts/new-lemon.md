---
title: git ssh 本机多账号
date: 2020-10-30 15:18:08
tags: git,ssh
---

# 本机如何使用ssh连接多个GitHub账号来生产多倍快乐

> github账号对应的邮箱账号个数

## 生成过程

1. 按照[generating new ssh key and adding it to the ssh-agent](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh)指示操作

    ```bash
    # 打开命令行
    # 查看现有的ssh keys
    ls -al ~/.ssh
    # 如果有以下文件中的一个，则代表待会创建的就是第二份的 ssh publick id_rsa
    id_rsa.pub id_ecdsa.pub id_ed25519.pub
    # Generating a new ssh key by your_email@example.com
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    # 命令行提示
    > Generating public/private rsa key pair.
    # 不要直接回车，否则会覆盖已有的，重新键入: 比如文件名这里改成了 id_dubble_rsa，记住这个文件名！
    > Enter a file in which to save the key (/Users/you/.ssh/id_rsa): /Users/you/.ssh/id_dubble_rsa
    # 接下来按照提示键入
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```

2. 将新的ssh key添加到 ssh-agent

    ```bash
    eval "$(ssh-agent -s)"
    > Agent pid 59566
    # 如果你是 mac 用户，还需要修改 ~/.ssh/config文件
    open ~/.ssh/config
    # 如果没有这个config文件，则手动生成一个
    touch ~/.ssh/config
    # 记事本会打开这个文件
    ```

    ```txt
    Host *
    AddKeysToAgent yes
    UseKeychain yes
    # id_dubble_rsa就是刚刚手动键入的文件名
    IdentityFile ~/.ssh/id_dubble_rsa
    ```

    ```bash
    # 最后一步添加
    ssh-add -K ~/.ssh/c
    ```

3. 将新的 ssh key 添加到GitHub账号

    ```bash
    # 复制对应的.pub文件内容
    pbcopy < ~/.ssh/id_dubble_rsa.pub
    # 打开 GitHub setting 页面的 ssh配置，将新的 key 复制进去即可
    # 查看是否连接正确
    ssh -vT git@github.com
    # 如果出现以下内容则成功！
    Hi your-github-name! You've successfully authenticated, but GitHub does not provide shell access.
    ```

## 使用之前

> 操作：1. 本地仓库推到远程；2. 远程仓库克隆下来 之前需要改下 `git config`

### 由于之前一直是单账号，所以现在要处理一下全局git配置

```bash
# 查看本机全局git配置，注意最后2行
# user.name='your-github-name'
# user.email='your-register-email'
git config -l
# 这2个值是第一个账号的值
# 现在如果进行新的github账号的各种仓库的操作，则需要进入到相关文件夹
# 使用一个空文件夹模拟过程
mkdir new-github-repo
cd new-github-repo/
# git 初始化，根据提示操作
git init
# 关键步骤：
# 清除全局配置
git config --global unset user.name your-old-github-name
git config --global unset user.email your-old-register-email-address
# 局部设置
git config --local user.name your-new-github-name
git config --global unset user.email your-new-register-email-address
# 最后再检查一下
git config -l
# 如果已经进行了 git commit msg操作，不用慌，还可以修改
# git commit --amend --reset-author
```

## 本地多个项目有不同的node版本管理

* nvm & avn 联合控制
* 项目中增加一个`.node-version` or `.nvmrc`文件即可,内容为该项目的node版本数字

```.nvmrc
12
```