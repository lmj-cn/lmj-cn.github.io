---
title: "记一种通过spoof Mac在Mac平台更改MAC地址的方法"
date: 2023-01-23T16:47:38+08:00
draft: true
tags: [OS,Mac]
categories: [OS]
---

## 介绍

修改电脑的MAC地址可实现针对以MAC地址为管理方式的破解，例如：

- 如果路由器以黑名单方式禁止某些设备链接WIFI
- 如果路由器针对某些设备做限速

> 修改的方式包括两种，一种为实际更换对应网络的物理网卡，例如从有线网卡更换为无线网卡或使用其他的外置网卡；另外一种方法为从操作系统层面更改从设备获取到的物理网卡的地址，实现对对应路由器的硬件信息欺骗。
> 本文介绍的spoof-mac就是在MacOS上可以自由更换网卡MAC地址的软件。

## spoof-mac安装

> spoof-mac的Github地址：[https://github.com/feross/SpoofMAC](https://github.com/feross/SpoofMAC)，介绍常见的多种环境下的安装

### brew安装

```shell
brew install spoof-mac
```

### pip安装

> 依赖pip，可选`pip`或`easy_install`

```shell
pip install SpoofMAC
easy_install SpoofMAC
```

### 源码安装

> 依赖python

```shell
git clone git://github.com/feross/SpoofMAC.git
cd SpoofMAC
python setup.py install
```

## 使用

```shell
➜  ns-cn.github.io git:(main) ✗ spoof-mac help
Usage:
    spoof-mac list [--wifi]
    spoof-mac randomize [--local] <devices>...
    spoof-mac set <mac> <devices>...
    spoof-mac reset <devices>...
    spoof-mac normalize <mac>
    spoof-mac -h | --help
    spoof-mac --version
```

> 以默认的en0为准，针对常见的随机生成MAC地址和还原默认的MAC地址做简单演示说明。针对MAC地址的变更需要重新连接WIFI才能生效。

### 随机生成MAC地址

```shell
sudo spoof-mac randomize en0
```

### 还原默认的MAC地址

```shell
sudo spoof-mac reset en0
```
