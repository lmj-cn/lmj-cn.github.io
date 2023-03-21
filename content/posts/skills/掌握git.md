---
title: 掌握git
date: 2023-03-07T10:38:56+08:00
draft: false
---

> 可视化学习网站：[国外站](https://learngitbranching.js.org/?locale=zh_CN)，[国内站](https://oschina.gitee.io/learn-git-branching/)
## 一、基本概念
### 1.1 SVN与GIT对比
#### SVN优点
* 较好的权限管理，可以精确控制每个目录的权限；
* 相对于git操作比较简单；
#### SVN缺点
* 集中式，如果中心服务器出现问题，所有人都不能正常干活，恢复也很麻烦，因为SVN记录的是每次改动的差异，不是完整文件；
* 分支功能弱
* 拷贝代码速度慢
* 必须联网使用
#### GIT优点
* 分布式，每个开发者的电脑上都有一个完整的仓库，不担心服务器问题；
* 不必联网使用，不联网也可以提交代码到本地仓库，可以查看以往的所有log，等到有网的时候，push到远程即可；
* 强大的分支管理功能
* Git的内容的完整性好: GIT的内容存储使用的是SHA-1哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。
#### GIT缺点
* 权限管理不够精细。
### 1.2 GIT入门
git工作目录下面的所有文件分为两种情况：以跟踪和未跟踪。
{{< mermaid >}}
sequenceDiagram
    participant 未跟踪(untracked)
    participant 未修改(unmodified)
    participant 已修改(modified)
    participant 已暂存(staged)
    未跟踪(untracked)->>已暂存(staged): Add the file
    未修改(unmodified)->>已修改(modified): Edit the file
    已修改(modified)->>已暂存(staged): Stage the file
    未修改(unmodified)->>未跟踪(untracked): Remove the file
    已暂存(staged)->>未修改(unmodified): Commit the file
{{< /mermaid >}}
### 1.3 安装
|系统|方式|
|:--:|:--:|
|Linux|`yum install git-all`或者`apt install git-all`|
|Mac|通过安装Xcode命令行工具安装|
|Windows|官网下载|
|源码|参考官方文档|
### 1.4 配置
|层级|文件位置|配置参数|
|:--:|:--:|:--:|
|系统级|`/etc/gitconfig`|`--system`|
|用户级|`~/.gitconfig`或者`~/.config/git/config`|`--global`|
|仓库级|`.git/config`||
配置用户身份
```bash
git config --global user.name "lmj-cn"
git config --global user.email "563935973@qq.com"
```
配置编辑器
```bash
git config --global core.editor emacs
```
检查个人配置
```bash
# 列举所有配置
git config --list
# 查看配置的用户名
git config user.name
# 查看config帮助文档
git help config
```
## 二、基础操作
### 2.1 git commit
创建一个新的提交记录
```bash
git commit -a "messages"
git commit -a -m "messages" # 自动把所有已跟踪（暂存和已修改）的文件
```
