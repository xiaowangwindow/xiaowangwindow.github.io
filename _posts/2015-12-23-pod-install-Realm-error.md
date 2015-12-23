---
layout: post
title: "Pod install Realm Error"
description: "core is not a symlink, downloading cor failed."
tags: [iOS, pod]
modified: 2015-12-23
image:
  feature: abstract-7.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

前阵子工作上的项目，打算用一下Realm这个神奇的数据库。在安装上，我碰到了很大的问题。

在pod install --no-repo-update时，总是出现以下错误:

```
Updating local specs repositories

Analyzing dependencies

Downloading dependencies

Installing Realm (0.96.2)

[!] /bin/bash -c 

set -e

sh build.sh cocoapods-setup

core is not a symlink. Deleting...

Downloading dependency: core 0.94.0

Downloading core failed. Please try again once you have an Internet connection.

```

看错误信息，大概是依赖需要的一个core文件无法下载。开了代理，还是出现这个错误。网上查了一下，还是有挺多人碰到这个问题的。刚开始以为是Realm新版本的问题，查了Realm的版本(0.96.2)、Pod的版本号(0.39.0)都很正常，更新specs repositories都没能解决。

后面看到了一个帖子:

> $ curl -f -L --verbose "https://static.realm.io/downloads/core/realm-core-0.94.0.tar.bz2" -o $TMPDIR/core_bin/core-0.94.0.tar.bz2

手动下载一个realm-core文件，然后执行pod install --no-repo-update 就解决问题。

[原帖链接](http://stackoverflow.com/questions/33725751/pod-install-doesnt-work-when-i-try-to-get-realm-0-96-2)
