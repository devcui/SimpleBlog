---
title: "在IPAD上调试Flutter应用"
date: 2021-12-05T11:30:03+00:00
tags: ["flutter","dart"]
series: []
categories: ["移动端"]
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

# 1.Flutter 环境

## 1.1 安装dart

```
brew tap dart-lang/dart
brew install dart
```

## 1.2 安装flutter

```
git clone https://github.com/flutter/flutter.git
```

配置环境变量

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
# 这里换成你自己clone的地址
export FLUTTER_HOME=$HOME/Sdks/flutter
export PATH=$PATH:$FLUTTER_HOME/bin
```

# 2.Android 环境

## 2.1 sdk

```
brew install --cask android-sdk
```

配置环境变量

```
# 后面的数字会灵活变动，使用brew info android-sdk查看路径
export ANDROID_SDK_HOME=/usr/local/Caskroom/android-sdk/4333796
export ANDROID_SDK_ROOT=/usr/local/Caskroom/android-sdk/4333796
export ANDROID_NDK_HOME=/usr/local/Caskroom/android-sdk/4333796/ndk
export PATH=$PATH:$ANDROID_SDK_ROOT/tools:$ANDROID_SDK_ROOT/platform-tools:$ANDROID_NDK_HOME:$ANDROID_SDK_ROOT/emulator:$ANDROID_SDK_HOME
```

## 2.2 avd

使用sdk-manager 安装avd，platform-tools等工具后便可使用了

# 3.xcode

## 3.1 code

```
sudo xcode-select --switch 
 /Applications/Xcode.app/Contents/Developer
        sudo xcodebuild -runFirstLaunch
```

## 3.2 cocoapods

```
brew cleanup -d -v
brew install cocoapods
```

此时可以使用 `flutter doctor`查看环境还有哪些问题

# 4.Project

![image.png](https://blog.eiyouhe.com/upload/2021/05/image-726f042a5ec644d98055c24fe8400c49.png)

打开VSCODE安装FLUTTER插件

![image.png](https://blog.eiyouhe.com/upload/2021/05/image-ff9352804b714f11adf9177968710a25.png)

输入`flutter: New`

选中第一个选项回车后建立项目

# 5.Run

*第一步: 请在vsocode右下角选中你的ipad设备*

![image.png](https://blog.eiyouhe.com/upload/2021/05/image-51d64d21451240c1afba62521343a0ff.png)

找到flutter的main文件点击run，报错如下

![image.png](https://blog.eiyouhe.com/upload/2021/05/image-a69778d07144498f828b5521ea1a6658.png)

此时使用xcode打开项目内的`ios`目录
![image.png](https://blog.eiyouhe.com/upload/2021/05/image-37da2bd2bc4349398eed5c446537dc66.png)

![image.png](https://blog.eiyouhe.com/upload/2021/05/image-c9d688570c834e4fb9ad1c7710029e7e.png)

```
# 然后 XCode 打开项目，找到 Flutter/Generated.xcconfig 文件
# 在文件最后加上
FLUTTER_BUILD_MODE=release
```

![image.png](https://blog.eiyouhe.com/upload/2021/05/image-460f6de150a24bcebf5bbcd2ea24e1bd.png)

# 6.Vscode

操作完以后返回Vscode点击Run就可以将flutter项目跑在ipad上了

