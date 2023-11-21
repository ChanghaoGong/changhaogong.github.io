---
layout: post
title:  "oh my zsh 配置美化shell界面"
date:  2018-11-15 14:51:01      
categories: shell
tags: shell zsh 
---

* content
{:toc}

**oh-my-zsh是一款zsh的插件管理器，zsh比bash功能丰富，但配置起来比较麻烦，omz可以省掉很多麻烦，让你的shell界面变得不那么枯燥，并且还有很多实用功能，比如优化tab补全纠错，记录常见历史目录和命令，命令语法高亮等等。**
# 安装zsh
```shell
sudo apt-get install zsh
zsh --version
chsh -s $(which zsh)
```
# 安装oh-my-zsh
```shell
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```
# 使用oh-my-zsh

zsh用户环境变量储存在 `~/.zshrc` 中，和`.bashrc`用法很相似，编辑该文件
```shell
vi ~/.zshrc
```
需要编辑的选项有
```shell
ZSH_THEME="ys"
plugins=(
  git
  history-substring-search
  zsh-syntax-highlighting
)
```
分布代表选用的主题和插件，这里我习惯用ys的主题，插件有git、历史补全搜索和语法高亮，其他插件可以自行探索

# 安装插件
```shell
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
编辑`~/.zshrc`在plugins里面添加这两个插件，然后`source ~/.zshrc`
最后设置历史命令补全
```shell
cd ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
source zsh-syntax-highlighting.zsh
cd ~/.oh-my-zsh/custom/plugins/zsh-history-substring-search
source zsh-syntax-highlighting.zsh
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
```
安装完毕，重新登录即可，界面如下
![](https://ws1.sinaimg.cn/large/006LiNKKly1fx8s2lnc4jj30mf0cngms.jpg)