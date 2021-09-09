---
title: Ubuntu 常见坑位指南 (基于18.04)
date: 2020-08-05 22:36:02
tags: 新知
categories: Linux
---

# 1. 键盘键位映射修改
众所周知，大小写锁定键处于黄金键位可使用频率很低，而使用频率很高的 `Ctrl` 却处在最左下角，尤其是 linux 经常需要使用 `Ctrl + Alt + T` 打开终端，手都快骨折撇断了，所以想把上述两个键位更换位置，于是有了这篇踩坑记。

## 首先，如果你想永久修改键位，则可直接更改系统文件
```
/usr/share/X11/xkb/keycodes/evdev
```

## 如果是临时交换，则可以编辑一个Xmodmap文件，并在终端里执行（建议抄答案）
```
vim ~/.Xmodmap
```
然后输入：
```
! Swap Caps_Lock and Control_L
!
remove Lock = Caps_Lock
remove Control = Control_L
keysym Control_L = Caps_Lock
keysym Caps_Lock = Control_L
add Lock = Caps_Lock
add Control = Control_L
```
（其他映射更改参考 [Xmodmap 的 man](https://www.x.org/archive/X11R6.8.1/doc/xmodmap.1.html)）

## 最后，让生效命令开机执行
打开 app **启动应用程序** 添加新的程序，并在命令中输入：
```
xmodmap ~/.Xmodmap
```
即可

<small>键位映射修改参考博客[钢镚的技术笔记](http://coinstechnote.blogspot.com/2018/05/ubuntu1804gnome.html)</small>
&nbsp;
&nbsp;
# chrome 字体修改
一直用 mac 浏览网页和 pdf ，配了一台跑代码的台式，Ubuntu 下的 chrome 字体看着实在难受，就好像自己眼花了一样，上网查了一下，找到问题和较为简单的解决方案。
## 问题
通常情况下，Ubuntu 18.04 下的程序遵循 Gnome Tweaks 下设置的字体渲染配置进行渲染，但 Chrome 不是。原因在于，Chrome 是读取 Linux 系统 fontconfig 配置的，默认情况下系统 fontconfig 中没有对字体全局 hint 进行配置，这导致 Chrome 始终以完全 hint 模式对字体进行渲染。思源黑体不自带有 hint 信息，因此经过完全 hint 之后会变得非常丑陋。
## 方法
在 /etc/fonts/ 下创建 local.conf 文件并写入如下内容全局关闭 hint：
```
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>
  <match target="font">
    <edit name="hintstyle" mode="assign">
      <const>hintnone</const>
    </edit>
  </match>
</fontconfig>
```

<small>参考 [Soulike 的博客](https://soulike.tech/article?id=34)</small>
&nbsp;
&nbsp;
# 命令行挂载与移除移动存储设备
## 挂载
首先查看系统磁盘设备信息，并找到待挂载设备文件 (Linux系统中所有设备均被视为文件)，这里以 `/dev/sdb1` 为例
```
sudo fdisk -l
```
创建挂载目录，一般选择挂载在 `/mnt` 目录下
```
sudo mkdir /mnt/mydisk
```
挂载
```
sudo mount /dev/sdb1 /mnt/mydisk
```
现在 `/mnt/mydisk` 下就是移动存储设备的根目录了。
## 卸载
```
sudo umount /mnt/sdb1
# 或
sudo umount /dev/sdb1
```
## 断电
对于移动硬盘，很多时候卸载之后并没有断电，直接拔非常危险(祭奠我第一张装满 1TB 资料的 WD 硬盘)，这时候可以使用断电指令，前提是先卸载。
```
udisksctl power-off -b /dev/sdb1
```
若系统较老，上述指令出错则可尝试以下命令：
```
udisks --detach /dev/sdb1
```

<small>参考 [stackexchange 问答](https://unix.stackexchange.com/questions/178638/eject-safely-remove-vs-umount)</small>