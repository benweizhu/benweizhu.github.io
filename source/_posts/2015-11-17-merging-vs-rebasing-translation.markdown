---
layout: post
title: "译 Merging vs Rebasing（未完成）"
date: 2015-11-17 21:58:52 +0800
comments: true
categories: 
---

原文地址：https://www.atlassian.com/git/tutorials/merging-vs-rebasing

git rebase命令对于初学git的人就是像是一种巫术，应该远离之，但是实际上如果使用适当，会让你的git生活变得更轻松。在本文中，我将会比较git rebase和相关的git merge命令，鉴别出在git工作流所有使用git rebase的可能性。

###基本概念

理解git rebase的第一件事情就是它和git merge是解决相同的问题。两个命令都是设计出来将一个分支上的代码变化和另一个分支上的代码变化进行集成-只是他们的做法不同。

考虑这样的场景，当你开始在专注于在某一个分支上进行一个新的feature（特性）开发时，另外一个团队在master分支上进行了新的提交。这导致一个forked的历史，这对于任何属性Git作为团队合作工作的人都是比较清楚的。

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/01.svg)

现在，假设在master分支上新提交的代码和你正在开发的特性相关。为了将这些新提交的代码吸收/包含（incorporate，在这里就翻译成合并，避免混淆），你有两个选择：merging（合并）和rebasing（复位基底/复基）。

###Merge选项

最简单的选择是将master分支合并（merge）到正在开发的特性分支上，如下：
{% codeblock lang:sh %}
git checkout feature
git merge master
{% endcodeblock %}

或者，你可以将他们压缩成一行：
{% codeblock lang:sh %}
git merge master feature
{% endcodeblock %}

这样就是在特性分支上创建了一个新的“merge commit”，他讲两个分支上的历史绑在了一起，就像下面你看到的分支结构：
![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/02.svg)

合并（merge）是很好的，因为他是一个非破坏性的操作。当前存在的分支是无论如何都不会改变的。这样就避免了所有因为rebase导致的坑（pitfall）。

另外一方面，这样意味着，在每次你吸收/包含（incorprate）特性分支会有外部（extraneous）来的合并提交。如果master分支比较活跃，这样机会污染你的特性分支的历史记录。当然也有减轻该问题影响的的高级git log选项，但是他就让其他开发人员很难理解项目的历史。

###Rebase选项

作为merge的另一种选择，又可以将特性分支rebase到master分支，通过下面的命令：
{% codeblock lang:sh %}
git checkout feature
git rebase master
{% endcodeblock %}
他将整个特性分支移动到了master分支的末梢（tip），有效的吸收/包含（incorprate）所有的提交到master分支。但是，和merge不同，rebase通过给原始分支上每一次提交创建全新的提交，从而重写了项目的历史。

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/03.svg)

Rebase的主要好处是你会得到一个更为清晰的项目历史。首先，它消除了所有git merge所需要的不必要的merge提交（merge commit）。其次，正如你在上面那张图中看到的，rebase也产生一个完美的线性项目历史-你可以从特性的末端一路查看到项目的开始而没有任何的fork。这样，使得你更容易使用像git log， git bisect和gitk来浏览项目（原文字面含义：在项目中导航（navigate））。

但是，为了这样淳朴的提交历史，有两点权衡/交易（trade-off，翻译为妥协更合适）：安全性和可追溯性（traceability）。如果你不遵循rebase的黄金准则，重写项目历史可能会给你的合作工作流带来潜在的悲惨结局。还有一点，虽然没有那么重要，rebase会丢失merge commit所提供的上下文-你无法看到上游的代码变化是合适包含（incorprate）到特性分支上的。

###Interactive Rebasing

Interactive rebasing gives you the opportunity to alter commits as they are moved to the new branch. This is even more powerful than an automated rebase, since it offers complete control over the branch’s commit history. Typically, this is used to clean up a messy history before merging a feature branch into master.

To begin an interactive rebasing session, pass the i option to the git rebase command:
{% codeblock lang:sh %}
git checkout feature
git rebase -i master
{% endcodeblock %}

This will open a text editor listing all of the commits that are about to be moved:
{% codeblock lang:sh %}
pick 33d5b7a Message for commit #1
pick 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
{% endcodeblock %}

This listing defines exactly what the branch will look like after the rebase is performed. By changing the pick command and/or re-ordering the entries, you can make the branch’s history look like whatever you want. For example, if the 2nd commit fixes a small problem in the 1st commit, you can condense them into a single commit with the fixup command:
{% codeblock lang:sh %}
pick 33d5b7a Message for commit #1
fixup 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
{% endcodeblock %}

When you save and close the file, Git will perform the rebase according to your instructions, resulting in project history that looks like the following:

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/04.svg)

Eliminating insignificant commits like this makes your feature’s history much easier to understand. This is something that git merge simply cannot do.

Rebase的黄金准则

一旦你理解了什么是rebase，最重要的就是学会合适不要用它。git rebase的黄金准则就是不要在公共的分支上使用。

比如，想象一下如果你将master分支rebase到特性分支：

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/05.svg)

这个rebase会将所有在master上的提交移动到feature分支的末端。问题是这还只是在你自己的仓库中发生了。所有其他的开发人员仍然工作在原来的master分支上。因为rebase导致了新的提交，Git会认为你的master分支的历史和其他人的偏离。

唯一同步两个master分支的办法就是将他们合并（merge）回去，导致一个额外的合并提交（merge commit），和两个一系列的提交的，这两个一系列提交的代码改变是相同的（一个是原始master分支上的，一个是你rebased的分支上的）。不用说，这是非常让人困惑的情况。

所以，在你开始做rebase的时候，永远问你自己一下，“是否有人也在用这个分支”，如果回答是，那么把你手移开键盘，开始思考一种非破坏性的方式来做改变(e.g., the git revert command，这个位置没搞懂)。

否则，你就可以安全的重写历史。

Force-Pushing

如果你想要rebased过的master分支push到远程服务器，Git就会阻止你这么做，因为他和远程的master分支是冲突的。但是，你可以强制push，通过 --force标志，比如：
{% codeblock lang:sh %}
# Be very careful with this command!
git push --force
{% endcodeblock %}

这样会重写远程master的分支来匹配在你仓库rebased过得master分支，这会让团队中的其他成员非常困惑。所以，只有当你非常清楚你要做什么的时候，你才可以非常小心的使用。

一种情况下，你应该使用force-pushing，就是在你push一个私有特性分支到远程仓库后，你执行一个本地清理。这就是像是在说“哎呀，我不是真的想要push特性分支的原始版本。就用当前这一个吧”，而且，很重要的是，没有其他人在原来的特性分支上工作。

##工作流指导

Rebase可以根据你的团队或多或少的包含到现有的Git工作流中。在本节，我们将看看rebase在一个特性开发的不同阶段所提供的好处。

在任何工作流中启用git rebase的第一步就是为每一个特性创建单独的分支。这将提供你必备的分支结构来安全的使用rebase：

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/06.svg)

##清理本地

将rebase包含到工作流中最好的办法之一就是清理本地，同步特性。通过周期性的执行一个interactive（交互性的）rebase，你可以保证在你的特性开发中的每次提交都是专注和有意义的（focused and meaningful）。这就让你在写你的代码的时候，不用担心变成被独立的提交 - 你可以在这样的事情发生之后，修好它。

当调用git rebase，对于新的base，你有两个选项：特性的父分支（比如：父分支），或者是特性分支的更早提交。我们在之前的Interactive Rebasing部分看到了第一种选项的例子。当你仅仅需要修复上几次提交的时候，第二个选项也是不错的选择。比如，下面的命令进行了一个只有最后三次提交提交的interactive rebase。

{% codeblock lang:sh %}
git checkout feature
git rebase -i HEAD~3
{% endcodeblock %}

通过指定HEAD~3新的base，你并没有真的移动你的分支，交互性的重写了跟在它后面三次提交。注意这并不会将上游的改变包含到特性分支。

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/07.svg)

如果你想要用这种方法重写整个特性，git merge-base命令可以用来找到特性分支原来的base。下面的命令将返回原来base的提交ID，你可以将它传递给git rebase：

{% codeblock lang:sh %}
git merge-base feature master
{% endcodeblock %}

interactive rebasing是向你的工作流中引入git rebase的最好方法。其他开发人员能够看到的，只有你的完成的产品，这样可以有非常干净，容易理解的特性分支历史。

但是，还是要说，这仅仅能够用在私有的特性分支上。如果你和其他开发人员工作在同一个特性分支上，那么这个分支就是共有的，那么你就不允许重写它的历史。

没有任何git合并方式来清除带有interactive rebase的本地提交。

###Incorporating Upstream Changes Into a Feature

In the Conceptual Overview section, we saw how a feature branch can incorporate upstream changes from master using either git merge or git rebase. Merging is a safe option that preserves the entire history of your repository, while rebasing creates a linear history by moving your feature branch onto the tip of master.

This use of git rebase is similar to a local cleanup (and can be performed simultaneously), but in the process it incorporates those upstream commits from master.

Keep in mind that it’s perfectly legal to rebase onto a remote branch instead of master. This can happen when collaborating on the same feature with another developer and you need to incorporate their changes into your repository.

For example, if you and another developer named John added commits to the feature branch, your repository might look like the following after fetching the remote feature branch from John’s repository:

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/08.svg)

You can resolve this fork the exact same way as you integrate upstream changes from master: either merge your local feature with john/feature, or rebase your local feature onto the tip of john/feature.


![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/09.svg)

Note that this rebase doesn’t violate the Golden Rule of Rebasing because only your local feature commits are being moved—everything before that is untouched. This is like saying, “add my changes to what John has already done.” In most circumstances, this is more intuitive than synchronizing with the remote branch via a merge commit.

By default, the git pull command performs a merge, but you can force it to integrate the remote branch with a rebase by passing it the --rebase option.

###Reviewing a Feature With a Pull Request

If you use pull requests as part of your code review process, you need to avoid using git rebase after creating the pull request. As soon as you make the pull request, other developers will be looking at your commits, which means that it’s a public branch. Re-writing its history will make it impossible for Git and your teammates to track any follow-up commits added to the feature.

Any changes from other developers need to be incorporated with git merge instead of git rebase.

For this reason, it’s usually a good idea to clean up your code with an interactive rebase before submitting your pull request.

###Integrating an Approved Feature

After a feature has been approved by your team, you have the option of rebasing the feature onto the tip of the master branch before using git merge to integrate the feature into the main code base.

This is a similar situation to incorporating upstream changes into a feature branch, but since you’re not allowed to re-write commits in the master branch, you have to eventually use git merge to integrate the feature. However, by performing a rebase before the merge, you’re assured that the merge will be fast-forwarded, resulting in a perfectly linear history. This also gives you the chance to squash any follow-up commits added during a pull request.

![Alt text](https://www.atlassian.com/git/images/tutorials/advanced/merging-vs-rebasing/10.svg)

If you’re not entirely comfortable with git rebase, you can always perform the rebase in a temporary branch. That way, if you accidentally mess up your feature’s history, you can check out the original branch and try again. For example:

{% codeblock lang:sh %}
git checkout feature
git checkout -b temporary-branch
git rebase -i master
# [Clean up the history]
git checkout master
git merge temporary-branch
{% endcodeblock %}
###Summary

And that’s all you really need to know to start rebasing your branches. If you would prefer a clean, linear history free of unnecessary merge commits, you should reach for git rebase instead of git merge when integrating changes from another branch.

On the other hand, if you want to preserve the complete history of your project and avoid the risk of re-writing public commits, you can stick with git merge. Either option is perfectly valid, but at least now you have the option of leveraging the benefits of git rebase.

