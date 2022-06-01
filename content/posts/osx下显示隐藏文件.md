---
title: osx下显示隐藏文件
date: 2015-12-31 11:23:50
tags:
 - osx技巧
---

前段时间入手了 MacBook ，虽然游戏性能真的不怎么样，但是用来工作，写代码什么的体验确实很棒，毕竟定位不同嘛。后来发现一个问题，就是要在 osx 中显示隐藏文件，并不能像 windows 一样方便。于是上网收集了一下。

<!-- more -->

1. 使用终端
在终端输入以下命令：
`defaults write com.apple.finder AppleShowAllFiles -bool YES`
然后确定。要重启 Finder 才能生效。继续输入一下命令：
`killall Finder`
点击确定。现在再打开 Finder ，就能看到隐藏文件了。
如果要恢复隐藏的话，在终端输入以下命令：
`defaults write com.apple.finder AppleShowAllFiles -bool NO`。
然后同样是重启 Finder ，就能恢复了。

2. 在 Finder 的服务中添加一个选项
在应用程序中找到 Automator ，打开之后，点击服务。在左侧的资源库中找到`Run Shell Script`（中文系统是：运行 shell 脚本），把它拖到右侧工作区中，将以下代码粘贴到输入框中：

```sh
STATUS=`defaults read com.apple.finder AppleShowAllFiles`
if [ $STATUS == YES ];
then
defaults write com.apple.finder AppleShowAllFiles NO
else
defaults write com.apple.finder AppleShowAllFiles YES
fi
killall Finder
```
    最后在上边的 Service receives 的下拉菜单中选择 ‘no input’ ，然后将工作流程保存为 ‘Toggle Hidden Files’ 。 现在打开 Finder 的服务菜单，就会看到 ‘Toggle Hidden Files’ 这一个选项了。

    ![hiddenFiles01](https://s1.ax1x.com/2022/06/01/XGfbWQ.png)

3. 在右键菜单中添加一下选项

    上一个方法是在 Finder 的服务菜单中添加选项，这个方法是在文件夹的右键菜单的服务菜单里面添加一下选项。
    前部分跟上一个方法一样，但这次我们在 Service receives 中选择 ‘文件或文件夹’，并在右边选中 Finder 。现在，就可以在 Finder 的右键菜单的服务中找到 ‘Toggle Hidden Files’ 了。

    ![hiddenFiles02](https://s1.ax1x.com/2022/06/01/XGfqzj.png)