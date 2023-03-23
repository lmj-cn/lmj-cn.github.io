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
> 已修改的文件也可以使用git add添加到暂存区，再修改后，可以用暂存区的文件还原（checkout），git add和git stage是同一个命令
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
### git commit
创建一个新的提交记录
```bash
git commit -a "messages" # 将暂存区内容提交到版本库
git commit -a -m "messages" # 自动把所有已跟踪（暂存和已修改）的文件提交
# 使用一次新的commit, 替代上一次提交
# 如果代码没有任何新变化, 则用来改写上一次commit的提交信息
git commit --amend -m "messages"
```
### git branch
创建新的分支
```bash
# 创建一个名为xx的新分支
git branch xx
# 切换到xx分支（HEAD指针指向xx分支的指针）
git checkout xx
```
### git merge 
合并分支
```bash 
# 把xx分支合并到当前分支
git merge xx
```
### git rebase
合并分支-取出一系列的提交记录，“复制”它们，然后在另外一个地方逐个的放下去。
```bash
# 把xx分支合并到当前分支
git merge xx 
```
> git rebase相对于git merge来说，可以创造更线性的提交历史。
## 三、进阶操作
### 分离HEAD
HEAD 是一个对当前检出记录的符号引用 —— 也就是指向你正在其基础上进行工作的提交记录。

HEAD 总是指向当前分支上最近一次提交记录。大多数修改提交树的 Git 命令都是从改变 HEAD 的指向开始的。
```bash
# HEAD -> main -> xx  变成  HEAD -> xx
git checkout xx # [xx]某次提交记录的id(哈希值)
#HEAD -> xx 变成 HEAD -> main -> xx 
git checkout HEAD # 或者git checkout main
```
### 相对引用
通过指定提交记录哈希值的方式在 Git 中移动不太方便，所以 Git 引入了相对引用。
* 使用 ^ 向上移动 1 个提交记录
* 使用 ~<num> 向上移动多个提交记录，如 ~3
```bash
git checkout HEAD^ # HEAD指针指向上一个提交记录。
git checkout HEAD~3 # HEAD指针指向上三个提交记录
git branch -f main HEAD~3 # 将main分支指向HEAD的第三级父提交，HEAD指针不动
```
### 撤销变更
```bash
git reset HEAD~1 # 本质上向上移动分支
```
> 在reset后,之前提交所做的变更还在，但是处于未加入暂存区状态。

```bash
git revert HEAD # 撤销当前提交的修改，并且产生一次提交记录
```
### 整理提交记录
```bash
git cherry-pick xx1 xx2 # 将xx1和xx2提交记录复制到当前分支下
git rebase -i HEAD~4 # 交互式rebase，在文本编辑器中调整这些提交，如调整顺序，删除，合并提交等。
```
### 标签
```bash
git tag xx1 xx # 将tag xx1指向xx提交。
git describe <ref> # 输出<tag>_<numCommits>_g<hash>，tag 表示的是离 ref 最近的标签， numCommits 是表示这个 ref 与 tag 相差有多少个提交记录， hash 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。
```
> HEAD无法指向tag
### 两个父节点
操作符 ^ 与 ~ 符一样，后面也可以跟一个数字。
但是该操作符后面的数字与 ~ 后面的不同，并不是用来指定向上返回几代，而是指定合并提交记录的某个父提交。
```bash
git checkout main^2
```
> 父提交与当前提交连接的直线是main^1, 曲线是main^2
## 四、远程操作
### git clone
本地创建一个远程仓库的拷贝
```bash
git clone https://github.com/lmj-cn/lmj-cn.github.io.git
```
### git fetch
git fetch 完成了仅有的但是很重要的两步:
* 从远程仓库下载本地仓库中缺失的提交记录
* 更新远程分支指针(如 o/main)
> 实际上将本地仓库中的远程分支更新成了远程仓库相应分支最新的状态。不会改变本地仓库的状态.

### git pull
git pull命令等于先后使用git fetch和git merge命令
```bash
git pull --rebase # git fetch + git rebase
```