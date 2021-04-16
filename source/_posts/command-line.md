---
title: 命令行环境
date: 2021-01-10 15:41:14
tags: vim, 信号, 终端多路复用
---

## Vim「作为编辑器的使用功能」

### 编辑模式

1. 正常：移动光标修改
2. 插入文本： 键入 **_i_** 进入插入模式
3. 替换文本：键入 **_R_** 进入替换模式
4. 可视化： 选中文本块，键入 **_v_** 进入可视化一般模式，**_V_** 进入可视化行模式，**_Ctrl+v_** 可视化块模式
5. 执行命令：键入 **_:_** 进入命令模式：

   - `:q` 退出关闭窗口
   - `:w` 保存（写）
   - `:wq` 保存并退出
   - `:e {file-name}` 打开要编辑的文件
   - `:ls` 显示打开的缓存
   - `:help {标题}` 打开帮助文档：
     - `:help :w` 打开`:w`命令的帮助文档
     - `:help w` 打开`w`移动的帮助文档

> 键入 `ESC` 切换为正常模式

## vim「作为编程语言」

### 移动-在缓存中导航

1. 基本移动：`hjkl(左下右上-逆时针)`
2. 词：`w`-下一个词，`b`-词初，`e`-词尾
3. 行：`0`行初，`^`-第一个非空格字符，`$`行尾
4. 屏幕：`H`-屏幕首行，`M`-屏幕中间，`L`-屏幕底部
5. 翻页：`Ctrl+u`上翻，`Ctrl+d`下翻
6. 文件： `gg`-文件头，`G`文件尾部
7. 杂项：`:{line number}<CR>` or `{line number}G`
8. 查找：`f{character}, t{character}, F{character}, T{character}`, (f/F = find, t/T = to, 小写是 forward, 大写是 backward)
   - find/to forward/backward {character} on the current line
   - `,` or `;`用于导航匹配
9. 搜索：`/{regex}`向后搜索，`n` or `N`用于导航匹配

### 编辑-动词

- `i` - insert mode
- `o`/`O` insert line below / above 向下、上插入行
- `d{motion}`删除{移动命令}，`dw`删除词,`d$`删除到行尾,`d0`-删除到行头
- `c{motion}`改变{移动命令}，`cw`改变词，`d{motion}`再`i`
- `x`删除字符，等同于`dl`
- `s`替换字符，等同于`xi`
- 可视化模式 + 操作
  - 选中文字，`d`删除或者`c`改变
- `u`撤销，`<C-r>`重做
- `y` to copy / “yank” (some other commands like d also copy)
- `p` 粘贴
- `~` 改变字符大小写

### 计数

> `3w`向前移动 3 个词，`5j`向下移动 5 行，`7dw`删除 7 个词

### 修饰语：用修饰语改变语义

> `i`, `a`表示在内部，在周围

- `ci(`改变当前括号内容
- `ci[`改变当前方括号内容
- `da'`删除一个单引号字符串，包括周围的单引号

## 进阶

> 宏(macro)：批处理，命令集合

1. 搜索&替换：`:s`替换
2. 多窗口：`:sp` or `:vsp`分割窗口，同一个缓存可以在多个窗口显示

   ```bash
   # 以下载的 vimrc 文件为例
   vim vimrc
   :sp
   # 此时会有2个窗口查看或编辑vimrc文件
   ```

3. 宏：`q{character}`开始在寄存器`{character}`中录制宏，`q`停止录制，`@`重放宏
4. 执行宏 `{次数}@{字符}`，执行宏`{次数}`次
5. 宏递归：

   1. `q{character}q`清除宏
   2. 录制该宏，用@{character}来递归调用该宏

   ```bash
   # 自动加末尾行注释宏**@t**
   touch new-practice-vimrc
   vim new-practice-vimrc
   # 录制宏t
   qt
   # i进入编辑模式,编辑器底部显示： --INSERT --Recording @t
   i
   # 输入通用行尾注释
   # esc退出到编辑模式,底部显示 --Recording @t
   # q停止宏录制
   q
   # :wq!保存退出编辑模式，结束
   ```

## vscode 配置 vim 插件

- 下载插件 Vim
- 配置 setting.json

```json
{
  "vim.easymotion": true,
  "vim.suround": true,
  "vim.incsearch": true,
  "vim.useSystemClipboard": true,
  "vim.useCtrlKeys": true,
  "vim.hlsearch": true,
  "vim.insertModeKeyBindings": [
    {
      "before": ["j", "j"],
      "after": ["<Esc>"]
    }
  ],
  "vim.normalModeKeyBindingsNonRecursive": [
    {
      "before": ["<leader>", "d"],
      "after": ["d", "d"]
    },
    {
      "before": ["<C-n>"],
      "commands": [":nohl"]
    }
  ],
  "vim.leader": "<space>",
  "vim.handleKeys": {
    "<C-a>": false,
    "<C-f>": false
  }
}
```

## 练习-见[missing-semester 文件夹](https://github.com/MillyTang/missing-semester)
