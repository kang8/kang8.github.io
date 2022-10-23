---
title: "如何在 kitty 和 neovim 中映射 command 键"
date: 2022-09-14T08:14:53+08:00
draft: false
---

## 背景

在看[最新（2022/9/12）](https://this-week-in-neovim.org/2022/Sep/12)的 This Week In Neovim 中发现了插件：[Hvassaa/sterm.nvim](https://github.com/Hvassaa/sterm.nvim)。~~一个非常简单小插件，能够快速打开/关闭 terminal。~~ 现以改用 lspsaga.nvim 中的 [Float terminal](https://github.com/glepnir/lspsaga.nvim/blob/5f17b9b7a8becc7d1593aae80d263ec936ae5ca7/README.md?plain=1#L459-L467)。

我个人在 VsCode 和 PHPStorm 中也经常用到该功能，顺便整合到 nvim 中。

问题来了，我使用的快捷键是 VsCode 自带的按键 [<cmd + j>](https://github.com/microsoft/vscode/blob/28e52a46fe8df0c924c881e438e124c05f171b9c/src/vs/workbench/browser/parts/panel/panelActions.ts#L325) 来切换 terminal。在 neovim 中 [<D-...>](https://github.com/neovim/neovim/blob/1e5daed67693f88d3ad515cf9e2d2f6d29c48a08/runtime/doc/intro.txt#L375) 表示映射 command 键，但设置 `<D-j>` 后，没有生效。

> 至于在 neovim 中为何 command 键（在 Windows/Linux 中又叫 Super）不起作用，我目前并没有答案。

我的环境，供读者参考：

* Hardware: Apple M1 MacBook Pro
* nvim version: v0.8.0
* terminal emulator: kitty 0.26.3

## 解决

### 不优雅的方法

首先在互联网上找到了这个[答案](https://github.com/kovidgoyal/kitty/issues/2706#issuecomment-886174507)。

在 kitty 配置文件中添加下列代码：

```conf
map cmd+j send_text all :execute "normal \<d-j>"\r
```

该解决方法是完完全全的按键映射，也就是按下 `<cmd + j>` 后，kitty 会将其映射为 `:execute "normal <d-j>"\r`。

该方式可以解决按键映射，但不够优雅。按下 `<cmd + j>` 后，会在命令行窗口显示该命令。

![direct key mapping](direct-key-mapping.png)

更要命的是该按键是全局的，无论是否在 neovim 中。比如在 shell 中无意触发该映射，会打断我的思路。

### 不是最优解，但可以接受

在寻找的过程中还存在一种声音：不使用 `<command>` 键，而是改用诸如 `<leader>` `<control>` `<shift>` 等 neovim 支持的很好的按键。

但我并不想妥协，在 MacOs 中 `<cmd + j>` 对我来说是一个非常顺手的键位，并且我也想搞清楚 `<command>` 键的映射方式，于是继续寻找答案。


最后在 V2EX 上找到 [skywind3000](https://github.com/skywind3000) 的回答[^skywind3000_answer]，再借助 kitty 对 [keyboard-protocol](https://sw.kovidgoyal.net/kitty/keyboard-protocol) 的支持，就能完成 command 的按键映射。

该做法的解决思路为：既然不能直接映射 command 键，那我退而求其次将我需要的按键映射成 neovim 支持很好的按键。对于 neovim 来说 F 按键就是非常好的选择。除了 \<F1>-\<F12> 外还有 \<F13>-\<F37> 一共 25 个虚拟按键，由于使用频率非常少，所以是映射 command 的最佳选择。

具体步骤，我是用 \<F14> 来映射：

1. 首先在 kitty 中运行内置命令 `kitty +kitten show_key` 中输入 `shift + <F2>` 拿到 \<F14> 的 key。

2. 在 kitty 的配置文件中，将 `cmd + j` 映射成 \<F14> 的 key

```conf
map cmd+j send_text all \x1b[1;2Q 
```

3. 在 nvovim 中将 \<F14> 映射成对应命令

```lua
vim.keymap.set('n', '<F14>', '<Cmd>Lspsaga open_floaterm<CR>', opts)
vim.keymap.set('t', '<F14>', [[<C-\><C-n><Cmd>Lspsaga close_floaterm<CR>]], opts)
```

这个做法虽然没有解决实际的问题，但比通过 kitty 直接映射的方式好用。

## 参考

* [如何使用 alacritty 在 neovim 上 map ctrl + shift + key](https://www.reddit.com/r/neovim/comments/mbj8m5/how_to_setup_ctrlshiftkey_mappings_in_neovim_and)
* [Fix Keyboard Input on Terminals](http://www.leonerd.org.uk/hacks/fixterms/)

[^skywind3000_answer]: https://www.v2ex.com/t/318520?p=1#r_3743712
