---
title: "Win下使用 GitBash下安装 rsync"
date: 2021-12-05T11:30:03+00:00
tags: ["git","git bash","rsync"]
series: []
categories: []
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

# 0.原因

不知道为什么MinGW下的sys1.0的rsync不好使，所以需要手动为gitbash安装一个

# 1.下载点

http://www2.futureware.at/~nickoe/msys2-mirror/msys/x86_64/

# 2.对于tar命令的zstd扩展

- zstd-1.4.7-1-x86_64.pkg.tar.xz

首先解压

`xz -d zstd-1.4.7-1-x86_64.pkg.tar.xz && tar -xvf zstd-1.4.7-1-x86_64.pkg.tar`

得到如下几个目录

- /usr 
- /usr/bin      这个目录存放了 zstd 的二进制脚本
- /usr/share    这个目录存放了man文档和license文件

将 /usr 直接覆盖到 Git 安装目录下

# 3.安装其他二进制包

- libzstd-1.5.0-1-x86_64.pkg.tar.zst
- libxxhash-0.8.0-1-x86_64.pkg.tar.zst
- rsync-3.2.3-1-x86_64.pkg.tar.zst

由于前面安装了 `zstd` 所以这里就可以使用 `tar` 来解压 `.zst`包了

`tar -I zstd -xvf libzstd-1.5.0-1-x86_64.pkg.tar.zst`
`tar -I zstd -xvf libxxhash-0.8.0-1-x86_64.pkg.tar.zst`
`tar -I zstd -xvf rsync-3.2.3-1-x86_64.pkg.tar.zst`


同样将`/usr`直接覆盖至 Git安装目录下

# 4.测试

重启Cmd/Gitbash

`rsync -v`
