---
layout: post
title: Sites
description: "常用站点收集"
tags: [me]
image:
  background: triangular.png
---
## 在 `Storyboard` 模式下，执行`dequeueReusableCellWithIdentifier:forIndexPath` 出现Crash, Log显示 `NSScanner nil string argument`，`libc++abi.dylib: terminate_handler unexpectedly threw an exception`

题目有点长，其实不复杂，原谅一下OC的函数风格。这个问题呢，知道答案之后也很简单，但是在我碰到的时候困扰了大半天。下面请耐心听我说完：

#### 复用出问题了？

看到这个问题，我首先想到了应该是复用出问题了。

试了一下，使用 `dequeueReusableCellWithIdentifier` 就不会crash，但是这样不保证返回cell，得自己从storyboard中初始化cell，我从来没这样做过，只从xib load过cell。

再试一下，使用 `registerClass:forCellReuseIdentifier` 在前面为该tableview注册cell，没有crash。看来真的是没法通过identifier找到cell造成的crash。但是这两种办法都没法正常显示cell，可见还是没有正确load cell。

看了网上的很多答案，大多是因为忘了在 `storyboard` 中写 cell 的 reuse identifier 造成的crash，这个一定得写！！我一直是知道到的，而且我这样做了，还是crash，这才是让我最百思不得其解的。

#### 吃个饭，静心想想

无奈，吃个饭，换个心情，从来再来。

**NSScanner nil string argument**
仔细看了一下苹果的 [官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSScanner_Class/#//apple_ref/occ/instp/NSScanner/scanLocation)，NSScanner是一个扫描字符串的抽象类，这个错误就是入参不对造成的，但是在哪里用到了NSScanner呢。

**libc++abi.dylib: terminate_handler unexpectedly threw an exception**
stackoverflow上看到了这个 [答案](http://stackoverflow.com/questions/25932033/libcabi-dylib-terminate-handler-unexpectedly-threw-an-exception-0-stack-tra)。 **恍然大悟！！**

我之前在用这个 storyboard 时，拖过一个 `outlet` ，然后我觉得名字取得不够好，就在代码里面修改了一下名字，我以为 `Xcode` 这么牛逼，肯定会帮我把 storyboard里面的关联一起改掉，但是它没有。**That all!**
