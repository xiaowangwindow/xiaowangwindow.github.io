---
layout: post
title: Mac最佳配置
description: "日常"
tags: [me]
image:
  background: triangular.png
---
## Mac 最佳配置
触控板三指支持

### Homebrew 安装
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
homebrew源配置
```
cd /usr/local
git remote set-url origin git://mirrors.ustc.edu.cn/homebrew.git
```
### 桌面应用安装

    brew cask search/install 


brew install iterm2
brew install 

### 个性配置
https://github.com/altercation/solarized (配色方案)

iterm2 配置
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh（oh-my-zsh配置）

vim配置
https://github.com/VundleVim/Vundle.vim/wiki （Vundle配置）

XCode 配置
alcatraz 管理插件安装
curl -fsSL https://raw.github.com/alcatraz/Alcatraz/master/Scripts/install.sh | sh

CocoaPods 安装
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
sudo gem install cocoapods

