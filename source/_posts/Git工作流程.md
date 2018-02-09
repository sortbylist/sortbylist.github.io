---
title: Git工作流程
date: 2018-02-09 11:41:02
tags: [Git,工作流]
---

## Git工作流程	

- 分类
  - Git flow
  - Github flow
  - Gitlab flow

- 共同点：

  **功能驱动**：需求是开发的起点，先有需求再有功能分支（feature branch）或者补丁分支（hotfix branch）。完成开发后，该分支就合并到主分支，然后被删除。

### Git flow

项目存在2个长期分支：

- 主分支master[存放对外发布的版本]
- 开发分支develop[日常开发的分支]

项目存在3种短期分支：

- 功能分支（feature branch）
- 补丁分支（hotfix branch）
- 预发分支（release branch）

一旦完成开发，它们就会被合并进`develop`或`master`，然后被删除。

优点：

- 清晰可控

缺点：

- 相对复杂，需要同时维护2个长期分支。大多数工具都将`master`当作默认分支，可开发在`develop`分支，导致要经常切换分支。
- 基于`版本发布`，目标是一段时间的产出一个新版本。如果是`持续发布`的项目，代码一有变动，就部署一次。这时`master`和`develop`分支差别不大，没必要维护2个长期分支。

### Github flow

Github flow是git flow的简化版，用于`持续发布`。

只有1个长期分支`master`，用起来非常简单。

官方推荐流程：

> 第一步：根据需求，从`master`拉出新分支，不区分功能分支或补丁分支。
>
> 第二步：新分支开发完成后，或者需要讨论的时候，就向`master`发起一个[pull request](https://help.github.com/articles/using-pull-requests/)（简称PR）。
>
> 第三步：Pull Request既是一个通知，让别人注意到你的请求，又是一种对话机制，大家一起评审和讨论你的代码。对话过程中，你还可以不断提交代码。
>
> 第四步：一旦你的Pull Request通过评审，并通过测试，就可以部署在生产上，如果出现问题，可以用现有的`master`分支代码进行回退。
>
> 第四步：如果部署在生产上的代码没有问题，那么你的Pull Request将被接受，合并进`master`。一旦合并（`merge`），Pull Request就会生成一条`commit`记录，方便以后搜索与查找。

优点：

- 简单
- 适用于`持续发布`的产品

缺点：

- `master`分支的更新必须与产品的发布一致。但有时候代码合并进`master`，并不代码它能立刻发布。比如苹果APP审核要时间，或者有些公司有发布窗口。导致线上版本落后于`master`分支。此时，需要新建一个`production`分支跟踪线上版本。

### Gitlab flow

>  Gitlab flow 是 Git flow 与 Github flow 的综合。它吸取了两者的优点，既有适应不同开发环境的弹性，又有单一主分支的简单和便利。它是 Gitlab.com 推荐的做法。

上游优先`upsteam first`：

- 即存在一个主分支`master`，它是所有其他分支的`上游`。只有上游分支采纳的代码变化，才能应用到其他分支。

持续发布：

- `master`分支：开发环境
- `pre-production`分支：预发环境
- `production`分支：生产环境

> 开发分支是预发分支的"上游"，预发分支又是生产分支的"上游"。代码的变化，必须由"上游"向"下游"发展。比如，生产环境出现了bug，这时就要新建一个功能分支，先把它合并到`master`，确认没有问题，再`cherry-pick`到`pre-production`，这一步也没有问题，才进入`production`。

> 只有紧急情况，才允许跳过上游，直接合并到下游分支。

版本发布：

- `master`分支：主分支
- `2-3-stable`分支：稳定版本分支 

> 只有修补bug，才允许将代码合并到这些分支，并且此时要更新小版本号（通过打tag方式）。

### 个人开发

建议Gitlab flow