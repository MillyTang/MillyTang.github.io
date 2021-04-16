---
title: profiling zsh shell scripts
date: 2021-03-04 18:40:05
tags: zsh shell, vscode Resolving Shell Environment is Slow
---

> vscode 右下角弹出 「Resolving Shell Environment is Slow」错误，令人心痛

## 瓶颈在于 nvm 的加载

每次项目下增加`.nvmrc` or `.node-version`文件时，shell加载就奇慢无比，看了官方提示，分别`vim ~/.bashrc` & `vim ~/.zshrc`后发现了要点： 瓶颈在nvm的加载处