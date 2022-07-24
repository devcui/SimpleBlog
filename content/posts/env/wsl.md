---
title: "WSL development environment under Win11"
date: 2021-12-31T11:59:59+00:00
tags: ["env"]
series: ["某男子开发的日常"]
categories: ["开发环境"]
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

# 1.Why to do that ?

Because in the eyes of a non-Microsoft developer, the environment of windows is not very friendly, but windows software is needed daily, so there are only two choices: vm or wsl. I personally think wsl is more convenient, so I will use wsl first this time.

The following are my target requirements:

- 1.Work and life are all in one system.
- 2.File/software isolation does not affect each other.
- 3.Complete development activities in wsl.
- 4.No installation development software in the real system.


# 2.Install Wsl

## preinstall

The [document](https://docs.microsoft.com/en-us/windows/wsl/install-manual) of Microsoft

- 1.Bios: Actvate the bios virtualization function.
- 2.Start WSL Service: input `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart` in the powershell(admin mode).
- 3.Start VM Service: input `dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart` in the powershell(admin mode).
- 4.Download Update: Download a [package](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) that update the kernel of linux.
- 5.Reboot: Reboot your Windows.
- 6.Install Update: Install the fiveth step that downloaded package.
- 7.WSL2: use `wsl --set-default-version 2` to open the second version.


## wsl

You can use `wsl --list --online` to find some distribution of linux and below is the release version I got through the command.

```
NAME            FRIENDLY NAME
Ubuntu          Ubuntu
Debian          Debian GNU/Linux
kali-linux      Kali Linux Rolling
openSUSE-42     openSUSE Leap 42
SLES-12         SUSE Linux Enterprise Server v12
Ubuntu-16.04    Ubuntu 16.04 LTS
Ubuntu-18.04    Ubuntu 18.04 LTS
Ubuntu-20.04    Ubuntu 20.04 LTS
```

Use `wsl --install --distribution Ubuntu/Debian/Ubuntu-20.04` to install the linux in your system and watting for the username and password to be entered when the installation is over.

## after install

Use `wsl --list -v` to watch your system

that's mine

```
  NAME            STATE           VERSION
* Ubuntu-20.04    Running         2
```

**input `ubuntu2004.exe config --default-user <your username>` to change the default user logined.**

As you have more and more software, wsl will become larger and larger, because wsl is installed in the C drive directory by default, so the external operating system will be more difficult to use. So you need to move the wsl system to a separate disk.

## move the wsl

- 1.Shutdown: Close your wsl terminal and use `wsl --shutdown` to shutdown wsl service.
- 2.Export: use `wsl --export Ubuntu/Debian/Ubuntu-20.04 D:\\wsl.tar` to export your wsl system.
- 3.Unregister: use `wsl --unregister Ubuntu/Debian/Ubuntu-20.04` to destory the system that in the wsl service.
- 4.Import: use `wsl --import Debian/Ubuntu/Ubuntu-20.04 E:\\wsl D:\\wsl.tar` to install system to the location you want.

You have a wsl2 linux system now.

# 3.A terminal software 

- 1.install [tabby](https://github.com/Eugeny/tabby) and set the `default profile for new tabs` equals `WSL`

# 4.Oh my zsh

Use tabby to open a new terminal tab and the following operations are all done in this tab.

- 1.Install zsh `sudo apt-get install zsh -y`.
- 2.Change the defualt env to zsh `chsh -s /bin/zsh`.
- 3.Install [oh-my-zsh](https://ohmyz.sh/) `sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
- 4.Download [font files](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k) and install it.
- 5.Set the tabby terminal software's defualt font `MesloLGS NF`.
- 6.Clone `https://github.com/romkatv/powerlevel10k.git` to `~/.oh-my-zsh/custom/themes` and replace `ZSH_THEME="powerlevel10k/powerlevel10k"` in `~/.zshrc`.
- 7.Close and reopen tabby terminal and complete the configuration according to the terminal prompt.

plugins 

- 1.[Here are some plugins](https://github.com/zsh-users). if you want to install them.
- 2.You can clone thme to `~/.oh-my-zsh/custom/plugins`
- 3.Configgure the plugins name in `~/.zshrc` like this

```
plugins=(
        git
        zsh-autosuggestions
        zsh-completions
        zsh-history-substring-search
        zsh-syntax-highlighting
)
```

# 5.Homebrew

`apt` softwares was so old so i use [homebrew](https://brew.sh/) to install softwares.

Input `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` in your terminal and follow the output logs to install that. there has some commands to use homebrew:

- 1.brew list: list all softwares your installed.
- 2.brew search name: search software.
- 3.brew install name: install software.

# 6.IDE

## 6.1 Visual Studio Code

Now you can download the vscode to connect the wsl to develop.

# 6.2 Neovim

If you want to use vim/neovim then you can input `brew install neovim`

- 1.Install python2,python3 and get pip2/pip3
- 2.`pip2 install neovim --user` `pip3 install neovim --user`

## Install `Vim-Plug`

- 1.`mkdir ~/.config/nvim -p`
- 2.`touch ~/.config/nvim/init.vim -p`
- 3.use `curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim` to install vim-plug
- 4.`mkdir ~/.config/nvim/vim-plug -p`
- 5.`touch ~/.config/nvim/vim-plug/plugins.vim`

Add the following to ~/.config/nvim/vim-plug/plugins.vim

```
" auto-install vim-plug
if empty(glob('~/.config/nvim/autoload/plug.vim'))
  silent !curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs
    \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  "autocmd VimEnter * PlugInstall
  "autocmd VimEnter * PlugInstall | source $MYVIMRC
endif

call plug#begin('~/.config/nvim/autoload/plugged')

    Plug 'scrooloose/NERDTree'

call plug#end()
```

You can see a plugin named NERDTree between in the `plug#beign` and  `plug#end`,So you can also config any plugin at here.

`source $HOME/.config/nvim/vim-plug/plugins.vim` to reload your env.


## Install Plugins

use `nvim` to start neovim editor and input `:PlugInstall` to install your plugins.

use `:q!` to exit neovim eidtor and reopen it with `nvim`.

use `:NERDTree` to show the filetree plugin.


## My neovim profile

git: https://github.com/devcui/neovim-configs

## End

I am learning vim from scratch, and I am only at an introductory level now. As I deepen my learning, follow-up vim articles will be published in the editor series. The configuration file will also be updated at any time, let us work hard together~
