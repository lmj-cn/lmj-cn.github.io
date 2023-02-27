---
title: "基于本站快速搭建自己的网站-New from this."
date: 2023-02-21T05:53:46Z
draft: false
---

> 本文基于仓库[lmj-cn/lmj-cn.github.io](https://github.com/lmj-cn/lmj-cn.github.io)和[ns-cn/ns-cn.github.io](https://github.com/ns-cn/ns-cn.github.io)使用Hugo和Github Pages搭建个人博客

## 一、依赖组件
|组件|说明|
|:---:|:---:|
|Github|建立账号|
|git|本地安装git，配置github账号和token|
|hugo|本地配置使用([官网](https://gohugo.io/)、[安装手册](https://gohugo.io/installation/))，注意需要安装ext版本|
## 二、核心步骤
{{< mermaid >}}
flowchart LR
    s1[克隆仓库]-->s2[初始化]
    s2-->s3[新建文章]
    s3-->s4[本地调试]
    s4-->s5[远程推送]
    s5-->s6[配置GithubPages]
    s6-->s7[访问测试]
{{</mermaid>}}

## 三、11