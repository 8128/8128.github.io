---
title: Mac终端agnoster主题配置
comments: true
toc: true
toc_number: false
copyright: true
mathjax: false
katex: false
hide: false
date: 2020-04-02 18:32:22
tags:
categories:
description: MacOS自带终端实在过于辣眼睛？iTerm2 + Solarized + oh-my-zsh + agnoster 洗眼！
cover: https://raw.githubusercontent.com/8128/PicGo/master/20200402185427.png
top_img:
keywords: Mac, MacOS, agnoster, iTerm2, iTerm, Solarized, zsh, oh-my-zsh
---

来源English version：[ZenLulz](https://gist.github.com/ZenLulz/c812f70fc86ebdbb189d9fb82f98197e)

最终效果:

![](https://raw.githubusercontent.com/8128/PicGo/master/20200402183141.png)

## 安装 iTerm2

iTerm2是一个不错的终端

使用 [Homebrew](https://brew.sh/)进行安装（没有homebrew官网有一键安装命令）:

```shell
$ brew cask install iterm2
```

## 用 oh-my-zsh 安装 zsh 

[Oh-My-Zsh](http://ohmyz.sh/) 是一个开源的，社区驱动的框架，用于管理您的ZSH配置。它捆绑了许多有用的功能，助手，插件，主题等牛逼哄哄的东西。安装：

用curl：

```shell
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

用wget:

```shell
$ sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

## 应用Solarized

### 把zsh的主题设置为agnoster

打开zsh设置文件

```shell
$ nano ~/.zshrc
```

在其中搜索

```
ZSH_THEME="robbyrussell"
```

将这行改为:

```
ZSH_THEME="agnoster"
```

退出保存

### 安装所需要的字体

我们需要为iTerm2提供特定的字体，以正确处理主题*agnoster*。为此，我们将安装[Powerline字体](https://github.com/powerline/fonts)。

```shell
# Cloning
git clone https://github.com/powerline/fonts.git

# Installation
cd fonts
./install.sh

# Clean up
cd ..
rm -rf fonts
```

### 应用刚刚安装的字体

打开iTerm2的preference（Ctrl +，)，依次单击“ *Profiles* ”选项卡和*“* *Text* ”选项卡，最后单击“Change font”按钮，**为Powerline**选择Meslo **LG S DZ Regular**，默认大小为**12pt** 。

### 配置iTerm2中终端颜色

打开iTerm2的首选项（Ctrl +，），依次单击“ **Profiles** ”选项卡和“**Colors**”，最后单击下拉列表“ *Color Presets...”*。选择项目*Solarized Dark*。

## 设置系统中iTerm2热键

可以使用系统级热键在屏幕上给定位置调用iTerm2。要启用该功能，请打开iTerm2的preference（Ctrl +，），然后单击“ *Keys* ”选项卡，然后选中“ *Show/hide iTerm2*”和“*system-wide hotkey*”，然后“Hotkey toggles a dedicated window with profile“。

## 在Finder右键菜单中添加 *New Terminal Here*

传送门 [TermHere](https://hbang.ws/apps/termhere/)