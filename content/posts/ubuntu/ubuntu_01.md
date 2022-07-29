---
title: "Ubuntu_01"
date: 2022-07-26T00:00:00-18:00
tags: ["linux","ubuntu"]
series: ["learning linux"]
categories: ["operating system"]
author: cui
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Create a topic to record the linux learning process."
math: true
cover:
    image: /pokemon/images/ubuntu/neofetch.png
    caption: "ubuntu-neofetch"
    alt: "ubuntu"
    relative: true
    responsiveImages: true
    hidden: false
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowCodeCopyButtons: true
ShowWordCount: true
ShowCanonicalLink: false
canonicalURL: "https://devcui.github.io/pokemon/posts/ubuntu/ubuntu-01"
CanonicalLinkText: "copy from"
robotsNoIndex: true
---

# 1. Version

- `cat /proc/version`
  
  - ```bash
    Linux version 5.15.0-41-generic (buildd@lcy02-amd64-105) (gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0, GNU ld (GNU Binutils for Ubuntu) 2.34) #44~20.04.1-Ubuntu SMP Fri Jun 24 13:27:29 UTC 2022
    ```

- `uname -a`
  
  - ```bash
    Linux gn 5.15.0-41-generic #44~20.04.1-Ubuntu SMP Fri Jun 24 13:27:29 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
    ```
  
  - ```bash
    gn% uname --help
    Usage: uname [OPTION]...
    Print certain system information.  With no OPTION, same as -s.
    
      -a, --all                print all information, in the following order,
                                 except omit -p and -i if unknown:
      -s, --kernel-name        print the kernel name
      -n, --nodename           print the network node hostname
      -r, --kernel-release     print the kernel release
      -v, --kernel-version     print the kernel version
      -m, --machine            print the machine hardware name
      -p, --processor          print the processor type (non-portable)
      -i, --hardware-platform  print the hardware platform (non-portable)
      -o, --operating-system   print the operating system
          --help     display this help and exit
          --version  output version information and exit
    ```

- `neofetch`
  
  - ```bash
                .-/+oossssoo+/-.               cui@gn 
            `:+ssssssssssssssssss+:`           ------ 
          -+ssssssssssssssssssyyssss+-         OS: Ubuntu 20.04.4 LTS x86_64 
        .ossssssssssssssssssdMMMNysssso.       Kernel: 5.15.0-41-generic 
       /ssssssssssshdmmNNmmyNMMMMhssssss/      Uptime: 12 mins 
      +ssssssssshmydMMMMMMMNddddyssssssss+     Packages: 2115 (dpkg), 12 (brew), 7 (snap) 
     /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Shell: zsh 5.8 
    .ssssssssdMMMNhsssssssssshNMMMdssssssss.   Resolution: 2560x1440 
    +sssshhhyNMMNyssssssssssssyNMMMysssssss+   WM: bspwm 
    ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Theme: Adwaita [GTK3] 
    ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Icons: Adwaita [GTK3] 
    +sssshhhyNMMNyssssssssssssyNMMMysssssss+   Terminal: kitty 
    .ssssssssdMMMNhsssssssssshNMMMdssssssss.   CPU: AMD Ryzen 7 5800X (16) @ 3.800GHz 
     /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/    GPU: AMD ATI 09:00.0 Device 73ff 
      +sssssssssdmydMMMMMMMMddddyssssssss+     Memory: 1984MiB / 31986MiB 
       /ssssssssssshdmNNNNmyNMMMMhssssss/
        .ossssssssssssssssssdMMMNysssso.                               
          -+sssssssssssssssssyyyssss+-                                 
            `:+ssssssssssssssssss+:`
                .-/+oossssoo+/-.
    ```
  
  - you should install it before using ` apt-get install neofetch`

# 2. After Install

- reset password
  
  - use `sudo passwd` to reset the password of `sudo`

- update your system
  
  - use `sudo apt-get update` to recache software list
  
  - use `sudo apt-get upgrade` to upgrade your system

# 3. Software

- driver
  
  - amdgpu
    - godo [admdriver](https://www.amd.com/zh-hans/support) and choose your graph card version
    - ![amd](/pokemon/images/ubuntu/amd.png)
    - `sudo apt install ./amdgpu-sudo apt install ./amdgpu-install_22.20.50200-1_all.deb`
    - if prompted for missing dependencies then you can fix it with `sudo apt-get install -f`
    - now you have a new command named `amdgpu-install`
    - `sudo amdgpu-install`

- network
  
  - clash
    - goto [clash for windows](https://github.com/Fndroid/clash_for_windows_pkg/releases) to download 
    - `tar -zxvf Clash.for.Windows-0.19.25-x64-linux.tar.gz `
    - ```bash
      gn% sudo apt install ./amdgpu-install_22.20.50200-1_all.deb 
      gn% amdgpu-install amdgpu-install_22.20.50200-1_all.deb 
      gn% amdgpu-
      gn% sudo amdgpu-install l
      gn% ls
       amdgpu-install_22.20.50200-1_all.deb  'Unconfirmed 36253.crdownload'
      gn% ta
      gn% ls
      amdgpu-install_22.20.50200-1_all.deb  Clash.for.Windows-0.19.25-x64-linux.tar.gz
      gn% tar -zxvf Clash.for.Windows-0.19.25-x64-linux.tar.gz 
      Clash for Windows-0.19.25-x64-linux/
      ......
      Clash for Windows-0.19.25-x64-linux/swiftshader/libEGL.so
      Clash for Windows-0.19.25-x64-linux/swiftshader/libGLESv2.so
      Clash for Windows-0.19.25-x64-linux/v8_context_snapshot.bin
      Clash for Windows-0.19.25-x64-linux/vk_swiftshader_icd.json
      ```
    - When you unzip it,you will get the files listed in the above list and usually i move these files to `/opt` directory
    - `sudo mv Clash\ for\ Windows-0.19.25-x64-linux/ /opt/Clash`
    - now you can open it with the binary script it provides `/opt/Clash/cfw` 

- input method
  
  - sogouinput
    - [Sogou input method linux-installation guide](https://shurufa.sogou.com/linux/guide)

- browser
  
  - google-chrome
    - goto download [chrome](https://www.google.cn/intl/zh-CN/chrome/) and install it with `sudo apt install google-chrome-stable_current_amd64.deb`

- editor
  
  - marktext
    - download [marktext](https://github.com/marktext/marktext/releases) and install it with`sudo apt install marktext-amd64.deb`

In addition to these there are `vscode`,`jetbrains`,`lunacy`,`cider`...wich are all 

`.deb` packages and you can also install them with `sudo apt install xxx.deb` and use `sudo apt-get install -f` to fix missing dependencies errors.
