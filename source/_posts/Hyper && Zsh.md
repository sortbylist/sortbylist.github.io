---
title: Hyper && Zsh
date: 2018-04-15 12:40:05
tags: [Hyper,Zsh]
---

> 参考：[在 windows10 安裝 bash& oh-my-zsh](https://medium.com/@nonegrame/%E5%9C%A8-windows10-%E5%AE%89%E8%A3%9D-oh-my-zsh-916105cf36f7)

### Win7 without wsl

- 下载hyper： [windows版本](https://releases.hyper.is/download/win)

- 中文显示问题 

  修改`.hyper.js` 里面 TermCSS 改成：`termCSS: '.wc-node{width: 16px !important}'`，或者`termCSS: '.wc-node.unicode-node{width: 1em}'`,后面一种修改更推荐

- hyper-material-theme

  修改`.hyper.js`以下内容：
  ```js
     plugins: ['hyper-material-theme'],

     colors: {..},

     MaterialTheme: {

     // Set the theme variant,

     // OPTIONS: 'Darker', 'Palenight', ''

     theme: 'Darker',

     // [Optional] Set the rgba() app background opacity, useful when enableVibrance is true

     // OPTIONS: From 0.1 to 1

     backgroundOpacity: '1',

     // [Optional] Set the accent color for the current active tab

     accentColor: '#64FFDA',

     // [Optional] Mac Only. Need restart. Enable the vibrance and blurred background

     // OPTIONS: 'dark', 'ultra-dark', 'bright'

     // NOTE: The backgroundOpacity should be between 0.1 and 0.9 to see the effect.

     vibrancy: 'dark'

     },
  ```
### Win10 with wsl

-  启用wsl
  - 程序和功能
  - 启用或关闭windows功能
  - 启用适用于Linux的windows子系统
- 应用商店搜索wsl下载ubuntu并配置账号 
- 配置hyper启动后登录wsl
  ```js
  shell: 'C:\Windows\System32\cmd.exe', <---------------- shell: '',
  
  shellArgs: ['--login', '-i', '/c wsl'], <----------------- shellArgs: ['--login'],
  ```

<!-- more -->

### Linux配置Zsh 

- 安装zsh 
  - 先安装curl和git
  - Install cURL：`sudo apt-get install curl`
  - Install Git
     ```bash
      sudo apt-get update
      sudo apt-get install git
      sudo apt-add-repository ppa:git-core/ppa
     ```
  - 然后安装zsh   

      ```bash
      sudo apt-get install zsh
      ```
  - 最后安装oh-my-zsh 
      ```bash
      curl -L https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh | bash
      ```

- 配置终端使用zsh 
  ```bash
   bash -c zsh
  ```
   或永久设置
   ```bash
   chsh -s /usr/bin/zsh
   ```

- 某些zsh主题需要额外fonts

   - ubuntu系统使用命令安装
     
     ```bash
     sudo apt-get install fonts-powerline
     ```

   - centos等

     ```bash
     git clone https://github.com/powerline/fonts.git --depth=1

     cd fonts

     ./install.sh
     ```
   - windows下载git库后直接右键安装，然后修改.hyper.js

     ```js
     // default font size in pixels for all tabs

     fontSize: 16,

     // font family with optional fallbacks

     fontFamily: '"Source Code Pro for Powerline", Menlo, "DejaVu Sans Mono", Consolas, "Lucida Console", monospace',
     ```
### zsh配置

   ```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
   ```
  修改`.zshrc`文件：`vi ~/.zshrc`

```
   plugins=(
       git
       zsh-syntax-highlighting
   )
```

### 配置bitvise ssh client的terminal 

- 在标题上右键-properties-font face-选择Source Code Pro for Powerline

