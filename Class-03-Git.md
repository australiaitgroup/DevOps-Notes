# Git 的应用

## 目录

- [DevOps 为什么要使用 Git](#devops-为什么要使用-git)
- [版本控制系统是什么](#版本控制系统是什么)
- [为什么要使用版本控制](#为什么要使用版本控制)
- [版本控制术语](#版本控制术语)
- [集中式与分布式版本控制](#集中式与分布式版本控制)
- [什么是 Git](#什么是-git)
- [第 0 步：设置 Git](#第-0-步设置-git)
  - [全局设置](#全局设置)
  - [本地设置](#本地设置)
  - [设置代码仓库](#设置代码仓库)
  - [忽略文件](#忽略文件)
  - [云端代码仓库](#云端代码仓库)
- [第 1 步：提交更改](#第-1-步提交更改)
  - [Git 工作原理](#git-工作原理)
  - [提交更改](#提交更改)
  - [撤销操作](#撤销操作)
  - [查看状态和历史](#查看状态和历史)
- [第 2 步：分支与合并](#第-2-步分支与合并)
  - [分支操作](#分支操作)
  - [合并分支](#合并分支)
  - [合并冲突](#合并冲突)
  - [合并与变基](#合并与变基)
  - [使用 Git 历史插件](#使用-git-历史插件)
- [第 3 步：远程协作](#第-3-步远程协作)
  - [连接远程仓库](#连接远程仓库)
  - [设置跟踪分支](#设置跟踪分支)
  - [从远程更新本地](#从远程更新本地)
  - [将本地更新推送到远程](#将本地更新推送到远程)
  - [同步](#同步)
  - [删除远程分支](#删除远程分支)
  - [拉取请求（PR）](#拉取请求pr)
  - [忽略文件](#忽略文件)
  - [关键点](#关键点)
  - [强制推送](#强制推送)
- [作业与挑战](#作业与挑战)

## DevOps 为什么要使用 Git

Git 是一种连接开发团队和运维团队的工具。通过 Git，开发人员可以将源代码提交，自动测试和部署过程中如果发现代码有问题无法通过要求，需要通过 Git 将代码打回去。DevOps 需要编写脚本和模板，管理复杂的部署流程，使用 Git 来管理这些文件和分支。

## 版本控制系统是什么

版本控制系统用于记录版本变更，Git 是一种版本控制工具。它管理源代码的版本，记录文本文件的修改，每次修改都会生成新的版本。

## 为什么要使用版本控制

- **撤销改动或回滚版本**：记录每次修改，形成时间线，允许回滚到之前的状态。
- **回溯历史**：团队合作时，可以查看历史记录，了解文件为何变成现在的样子。
- **协同合作**：团队成员可以同时修改不同段落，Git 可以帮助合并不同的修改。
- **备份**：Git 记录最新版本，当本地内容丢失时，可以拉取最新内容。

## 版本控制术语

- **Repository**：项目仓库
- **Diff**：两个状态之间的差异
- **Commit**：项目在某个时间点的快照
- **Branch**：与主分支并行的修改
- **Merge**：将一个分支的修改引入到另一个分支
- **Clone**：下载项目的本地副本
- **Fork**：对他人项目的版本进行修改

## 集中式与分布式版本控制

- **集中式版本控制**：依赖于中央服务器，单点故障风险高，远程提交慢。
- **分布式版本控制**：没有单点故障风险，每个开发者有完整副本，可以离线提交，分支操作快。

## 什么是 Git

Git 是一种分布式版本控制系统，设计用于跟踪软件开发中的源代码更改。它支持分布式、非线性的工作流，目标是速度和数据完整性。

## 第 0 步：设置 Git

Git 是一个命令行工具，首先确认 Git 是否已经安装，输入 `git --version`。

## 全局设置

```sh
git config --global user.name "your name"
git config --global user.email "your email address"
git config --global core.editor "code --wait"
```

使用 `--global` 设置全局配置，不使用 `--global` 则只影响当前仓库。

## 本地设置

不加 `--global` 设置本地配置，优先级高于全局配置。

## 设置代码仓库

Git 只能管理文本文件，不能管理复杂文件（如 Word）。

### 初始化仓库

```sh
mkdir learngit
cd learngit
pwd
/Users/jonny/learngit
git init
```

或使用 `git clone` 克隆现有仓库。

```sh
git clone <repository-url>
```

## 忽略文件

创建 `.gitignore` 文件列出不需要 Git 追踪的文件和文件夹。

更多 `.gitignore` 模板：[GitHub .gitignore](https://github.com/github/gitignore)

## 云端代码仓库

创建 GitHub 仓库：[GitHub 新建仓库](https://github.com/new)

## 第 1 步：提交更改

### Git 工作原理

Git 提交分两步：`git add` 和 `git commit`。

```sh
git add <file>
git add .  # 提交所有文件到暂存区
git commit -m "commit message"
```

### 提交更改

提交信息要有意义，回顾历史时可以清楚理解每次提交的用途。

### 撤销操作

- **撤销最后一次提交**：`git commit --amend`
- **删除文件**：`git rm <file>`
- **清理工作区**：`git clean -f`
- **暂存修改**：`git stash`
- **应用暂存**：`git stash apply`
- **弹出暂存**：`git stash pop stash@{0}`

### 查看状态和历史

常用命令：

```sh
git status
git diff
git log --all --decorate --oneline --graph
```

## 第 2 步：分支与合并

### 分支操作

- 创建分支：`git branch <branch-name>`
- 切换分支：`git checkout <branch-name>`
- 删除分支：`git branch -d <branch-name>`
- 创建并切换分支：`git checkout -b <branch-name>`

### 合并分支

```sh
git checkout main
git merge <branch-name>
git branch -d <branch-name>
```

### 合并冲突

如果不想解决冲突：

```sh
git merge <branch-name> --abort
git merge <branch-name> --overwrite-ignore
git merge <branch-name> --no-overwrite-ignore
```

### 合并与变基

- 合并：`git merge <branch-name>`
- 变基：`git rebase <branch-name>`

### 使用 Git 历史插件

使用 VSCode 的 Git History 插件可视化历史记录：[Git History Plugin](https://git-school.github.io/visualizing-git/)

## 第 3 步：远程协作

### 连接远程仓库

手动建立或删除仓库联系：

```sh
git remote add <name> <url>
git remote rm <name>
git remote rename <old-name> <new-name>
```

### 设置跟踪分支

```sh
git branch -u <name>/<branch>
```

### 从远程更新本地

```sh
git fetch   # 检查本地和远程仓库是否相同
git pull
```

### 将本地更新推送到远程

```sh
git push <name> <branch>
git push <name> <local_branch>:<remote_branch>
```

### 同步

VSCode 中的同步操作包括 pull 和 push。

### 删除远程分支

```sh
git push -d <name> <branch>
git push <name> :<branch>
```

### 拉取请求（PR）

拉取请求（PR）允许其他人查看和评论你的修改。在 GitHub 上创建 PR：

- 创建分支
- 提交更改
- 打开拉取请求
- 讨论和审查
- 合并和部署

PR 的好处：

- 同行评审
- 充分测试，稳定性更好
- 减少冲突
- 持续交付
- 责任更明确

PR 合并策略：

- 创建分支
- 提交更改
- 打开拉取请求
-

讨论和审查

- 合并和部署

## 忽略文件

创建 `.gitignore` 文件列出不需要 Git 追踪的文件和文件夹。

## 关键点

- 永远不要忘记 `.gitignore`
- 在 PR 前解决冲突
- 尽量将同一功能或 Bug 修复的代码放在一个提交中
- 确保每次提交不会破坏构建流水线，代码风格检查和单元测试通过

## 强制推送

```sh
git push -f
git push <name> +<branch_name>
git push --force-with-lease
```

## 作业与挑战

- [Git 练习 1](https://github.com/JiangRenDevOps/DevOpsLectureNotesV6/blob/master/WK2_Git/git_exercise_1.md)
- [Git 练习 2](https://github.com/JiangRenDevOps/DevOpsLectureNotesV6/blob/master/WK2_Git/git_exercise_2.md)
- [学习 Git 分支](https://learngitbranching.js.org/)
