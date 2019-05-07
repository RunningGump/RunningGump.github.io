---
title: Tmux的基本使用
date: 2019-05-07 15:10:23
categories: Tmux
tags: 
   - Tmux 
---

# Tmux是什么

Tmux是一个优秀的终端复用软件，可以在一个终端窗口中运行多个终端会话。不仅如此，你还可以通过Tmux使终端会话运行于后台或是按需接入、断开会话。

使用Tmux连接到服务器可以解决很多令人头疼的问题：

- 想同时打开多个目录的时候，不得不开多个终端来回切换
- 运行一个脚本，服务器断掉失联之后当前进程会被服务器无情杀掉
- 每次ssh到服务器都要重新切到工作目录，打开工作进程等，无法保存之前的工作记录
- …...

# 安装Tmux

## 在Mac OS中上安装

- 安装HomeBrew

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- 安装Tmux

```bash
brew install tmux
```

## 在Ubuntu中安装

在终端输入如下命令：

```bash
sudo apt-get install tmux
```

# Tmux的配置文件

如何对Tmux的配置文件进行编写这里不做介绍，自行百度。这里直接给出网上大佬已经写好的配置文件的[Github地址](https://github.com/gpakosz/.tmux)。

下载方法（逐条执行）：

```bash
cd ~
git clone https://github.com/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```

下载完后，关闭所有终端，重启Tmux配置文件就生效了。有关该配置文件的详细特性，去看这位大佬的Github。这里仅列出几点重要的：

- Ctrl-a 和 Ctrl-b都可以作为Tmux的前缀prefix
- 最大化窗格为一个新的窗口 <prefix> +  (再按一次就又返回原来的布局了)
- 鼠标模式切换 <prefix> m

# Tmux的基本结构

| 单元模块 | 描述                               |
| -------- | ---------------------------------- |
| server   | 服务器，一个服务器可以包含多个会话 |
| session  | 会话，一个会话可以包含多个窗口     |
| window   | 窗口，一个窗口可以包含多个窗格     |
| panel    | 窗格                               |

# Tmux基本操作

基本操作无非是对会话、窗口、窗格进行管理，包括创建、关闭、重命名、连接、分离、选择等。Tmux默认的快捷键前缀是**Ctrl+b**(下文用**prefix**指代)，按下前缀组合键后松开，再按下命令键进行快捷操作，

## 会话管理(session)

### 常用命令

```bash
tmux new	# 创建默认名称的会话（在tmux命令模式使用**new**命令可实现同样的功能，其他命令同理，后文不再列出tmux终端命令）
tmux new -s session-name	# 创建名为session-name的会话 (常用)
tmux ls	# 显示会话列表
tmux a	# 连接（attach）上一个会话
tmux a -t session-name	# 连接指定会话
tmux rename -t s1 s2	# 重命名会话s1为s2
tmux kill-session	# 关闭上次打开的会话
tmux kill-session -t s1	# 关闭会话s1
tmux kill-session -a -t s1	# 关闭除s1外的所有会话
tmux kill-server	# 关闭所有会话
```

### 常用快捷键

```bash
prefix s	# 列出会话
prefix $	# 重命名会话
prefix d	# 离开当前会话
prefix D	# 离开指定对话
```

## 窗口管理(window)

```bash
prefix c	# 创建一个新窗口 (常用)
prefix ,	# 重命名当前窗口
prefix &	# 关闭当前窗口
prefix 0~9 	# 选择编号0~9对应的窗口
prefix w	# 列出所有窗口，可进行切换
prefix n	# 进入下一个窗口(next)
prefix p	# 进入上一个窗口(previous)
prefix l	# 进入之前操作的窗口
prefix .	# 修改当前窗口索引编号
prefix '	# 切换至指定编号(可大于9)的窗口
prefix f	# 根据显示的内容搜索窗格
```

## 窗格管理(pane)

```bash
prefix %	# 水平方向创建窗格
prefix "	# 垂直方向创建窗格
prefix Up|Down|Left|Right	# 根据箭头方向切换窗格
prefix x	# 关闭当前窗格
prefix q	# 显示窗格编号
prefix o	# 顺时针切换窗格
prefix Ctrl+o	# 逆时针切换窗格
prefix }	# 与下一个窗格交换位置
prefix {	# 与上一个窗格交换位置
prefix space(空格)	# 重新排列当前窗口下的所有窗格
prefix !	# 将当前窗格置于新窗口
prefix t	# 在当前窗格显示时间
prefix z	# 放大当前窗格(再次按下将还原)
prefix i	# 显示当前窗格信息
```

# 参考资料

1. [手把手教你使用终端复用神器Tmux](https://zhuanlan.zhihu.com/p/43687973)
2. [Tmux 速成教程：技巧和调整](http://blog.jobbole.com/87584/)