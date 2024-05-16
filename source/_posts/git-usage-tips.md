---
title: Git 使用小技巧
date: 2024-05-16 16:26:53
tags: [git, version-control, ChatGPT]
---

# git rebase 和 git merge 有什么区别，请简单解释下，以及需要遵守的原则

`git rebase` 和 `git merge` 都是用于合并不同分支上的更改的 Git 命令，但它们以不同的方式执行此操作，影响历史的结构和清晰度。

### git merge

- **合并提交**：`git merge` 会创建一个新的“合并提交”来将两个分支的更改结合起来。
- **非线性历史**：合并会保留分支的历史记录，因为它不会改变现有的提交历史。这意味着历史是非线性的，可以清楚地看到项目历史中的分支和合并点。
- **简单易懂**：对于不熟悉 Git 的人来说，合并可能更加直观，因为它保留了一个明确的分支和合并的历史记录。

### git rebase

- **重写历史**：`git rebase` 会将一个分支上的更改“重新播放”在另一个分支的顶部。这通常涉及更改提交的基础（即它们在历史中的位置）。
- **线性历史**：重建创建一个更干净、更直线的提交历史，因为它看起来就像所有的更改都是按顺序一个接一个地发生的，即使它们原本是在不同的分支上开发的。
- **冲突解决**：在 rebase 过程中，可能需要解决与基础分支上的新提交相关的冲突。这可能比合并时更频繁，因为每个提交都要重新应用。

### 需要遵守的原则

- **不要在公共分支上 rebase**：如果你的分支是别人也在用的（例如 `main` 或 `develop` 分支），那么不要使用 rebase。这会改变历史，对其他人来说可能会造成混乱。
- **合并公共分支，rebase特性分支**：通常，合并用于将更改从一个公共分支（如 `main`）合并到另一个公共分支（如 `release` 分支）。而 rebase 通常用于特性分支，以更新它们或准备合并到主分支。
- **清晰的历史**：如果你想要一个干净、线性的历史，那么 rebase 是一个不错的选择。如果你想保留完整的历史和分支上下文，那么合并可能更合适。
- **团队协作**：在团队环境中，确保所有成员都理解你们是合并还是 rebase，以及何时使用它们，以避免混乱。
- **安全操作**：在执行 rebase 之前，最好创建一个新的分支或确保你的更改已经推送到远程仓库，以防万一需要恢复原始分支。

使用这些命令时，考虑你的工作流程、团队的偏好以及你想要的提交历史的结构。在某些情况下，你可能会根据具体情况交替使用这两个命令。


# git 如何将 project 下面的默认分支由 main 改为 master

在Git中，更改项目的默认分支需要几个步骤。首先，你需要在本地仓库创建新的分支（如果还没有的话），然后推送到远程仓库，最后在远程仓库（比如GitHub, GitLab, Bitbucket等）的设置中更改默认分支。

下面是在命令行中执行这个操作的步骤：

1. **在本地创建新的`master`分支（如果还没有的话）**:
   
   如果你还没有`master`分支，你可以从`main`分支创建一个：

   ```sh
   git branch master main
   ```

2. **将新的`master`分支推送到远程仓库**:

   ```sh
   git push origin master
   ```

3. **在远程仓库设置中更改默认分支**:

   - 对于GitHub，你需要去仓库的 "Settings" -> "Branches" -> "Default branch"，选择`master`作为默认分支。
   - 对于GitLab，你需要去仓库的 "Settings" -> "Repository" -> "Default Branch"，选择`master`。
   - 对于Bitbucket，你需要去仓库的 "Repository settings" -> "Branches" -> "Main branch"，选择`master`。

4. **（可选）删除旧的`main`分支**:

   如果你想要删除旧的`main`分支，你可以这样做：

   - 首先确保你不在`main`分支上：

     ```sh
     git checkout master
     ```

   - 删除本地的`main`分支：

     ```sh
     git branch -d main
     ```

   - 删除远程的`main`分支：

     ```sh
     git push origin --delete main
     ```

请注意，更改默认分支可能会对其他开发者和正在进行的工作产生影响，因此在执行这些操作之前，最好通知所有相关人员。此外，如果你的仓库已经配置了CI/CD或其他自动化系统，你可能需要更新它们以反映新的默认分支。


# git 如何撤销 develop 修改，保持和master 同步

在 Git 中，如果你想要将 `develop` 分支的状态回退到与 `master` 分支一致的状态，你可以使用几种不同的方法。下面是一些常见的做法：

### 方法一：重置（Reset）

如果你想要丢弃 `develop` 分支上所有不在 `master` 分支上的提交，你可以使用 `reset` 命令。首先，确保你已经拉取了最新的 `master` 分支的代码：

```bash
git checkout master
git pull origin master
```

然后，切换回 `develop` 分支，并使用 `reset` 命令来重置它：

```bash
git checkout develop
git reset --hard origin/master
```

这会将 `develop` 分支的头指针移动到 `master` 分支的当前位置，并且丢弃 `develop` 分支上 `reset` 之后的所有更改。

### 方法二：合并（Merge）

如果你想要保持 `develop` 分支上的提交历史，并且将其与 `master` 同步，你可以使用 `merge` 命令：

```bash
git checkout develop
git merge master
```

如果 `master` 分支上有 `develop` 分支上没有的提交，这些提交会被合并到 `develop` 分支上。如果 `develop` 分支上有一些额外的提交，它们将保持不变。

### 方法三：变基（Rebase）

如果你想要重新排列 `develop` 分支上的提交，使其看起来像是基于当前 `master` 分支的最新提交，你可以使用 `rebase` 命令：

```bash
git checkout develop
git rebase master
```

这将会把 `develop` 分支上的所有更改应用在 `master` 分支的顶端。这个过程可能会产生冲突，需要手动解决。

### 方法四：创建一个新的分支

如果你不想在 `develop` 分支上直接操作，你可以创建一个新的分支来替代当前的 `develop` 分支：

```bash
git checkout master
git branch -D develop  # 删除旧的 develop 分支
git checkout -b develop  # 基于 master 创建新的 develop 分支
git push -f origin develop  # 强制推送新的 develop 分支到远程仓库
```

请注意，强制推送 (`git push -f`) 可能会影响其他协作者，因为它会重写远程仓库的历史。在使用强制推送之前，请确保与你的团队成员沟通。

在执行上述任何操作之前，建议先备份你的分支，以防不小心丢失数据。使用 `git branch` 命令可以很容易地创建一个新的备份分支：

```bash
git checkout develop
git branch backup_develop  # 创建 develop 的备份
```

这样，如果出现任何问题，你可以使用备份分支来恢复你的状态。