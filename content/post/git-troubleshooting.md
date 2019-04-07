---
title: "Git Troubleshooting"
date: 2019-04-07T21:52:09+08:00
draft: false
---

# Q1. 相继执行 1) `git rebase master` 2) `git rebase --skip` 3) `git rebase --abort` 后，没有回到 1) 之前的状态，而是呈现 2) 之后的状态，有几个提交不见了。怎么才能恢复到 1) 之前？

虽然当前分支由于某种原因缺失了几个 commit, 但是 git 本身并不会删除 commit, 它们还是会被记录下来。只要找到对应的 sha1 hash, 就有办法恢复。具体见 Q2.

[这篇文章][1]详细叙述了上述问题的发生和解决过程。

// TODO figure out how `rebase --skip` works

# Q2. commit “丢了”怎么找回？

“丢失”原因可能是不小心执行了 `git reset --hard` 、误删了一个分支，或者是不恰当的 rebase 等。

## Solution 1

`git reflog`

官方文档描述：“当你在工作时， Git 会在后台保存一个引用日志(reflog)，引用日志记录了最近几个月你的 HEAD 和分支引用所指向的历史。使用 git reflog 可以查看引用日志。”

Variation:

git show HEAD@{5} // 查看仓库中 HEAD 在五次前的所指向的提交

git show master@{yesterday} // 查看 master 分支昨天指向哪个提交

git log -g // 查看类似于 git log 输出格式的引用日志信息

## Solution 2

`vim .git/logs/HEAD`

// TODO .git/logs 记录的内容与 reflog 有何异同？

# Q3. 高效解决冲突的办法？

以下步骤摘自 stackoverflow 的[一则回答][2]——

Try: `git mergetool`

Below is the sample procedure to use `vimdiff` for resolve merge conflicts.

**Step 1**: Run following commands in your terminal

    git config merge.tool vimdiff
    git config merge.conflictstyle diff3
    git config mergetool.prompt false

This will set vimdiff as the default merge tool.

**Step 2**: Run following command in terminal

    git mergetool

**Step 3**: You will see a vimdiff display in following format 

      +----------------------+
      |       |      |       |
      |LOCAL  |BASE  |REMOTE |
      |       |      |       |
      +----------------------+
      |      MERGED          |
      |                      |
      +----------------------+
These 4 views are 

> LOCAL – this is file from the current branch  

> BASE – common ancestor, how file looked before both changes 

> REMOTE – file you are merging into your branch 

> MERGED – merge result, this is what gets saved in the repo

You can navigate among these views using `ctrl+w`. You can directly reach MERGED view using `ctrl+w` followed by `j`.

**Step 4**. You could edit the MERGED view the following way 

If you want to get changes from REMOTE

    :diffg RE  

If you want to get changes from BASE

    :diffg BA  

If you want to get changes from LOCAL

    :diffg LO 

**Step 5**. Save, Exit, Commit and Clean up

`:wqa` save and exit from vi

`git commit -m "message"`

 `git clean` Remove extra files (e.g. *.orig) created by diff tool.

# Q4. `git rebase master` 时，由于当前分支对一处代码块的多次修改，导致要反复解决冲突，怎么办？

可以先把当前分支的多次修改整合为一个，再 rebase master, 这样就只需要 apply 一次修改、解决一次冲突。用的还是 `rebase` 命令，只不过参数不是 branch name, 而是 commit hash. 

假如当前分支的最近 5 次提交互相存在冲突，那么就执行：

`git rebase --interactive HEAD@{6}` // HEAD@{6} 代表上述 5 次冲突之前的 commit, 可以理解为“坐标原点”，基于这个点开始，整合之后的所有提交

# Reference

- [怎么查看“丢了”的 commit](https://stackoverflow.com/questions/4786972/get-a-list-of-all-git-commits-including-the-lost-ones)
- [Use gitk to understand git](https://lostechies.com/joshuaflanagan/2010/09/03/use-gitk-to-understand-git/)
- [Git 合并多个commit](https://segmentfault.com/a/1190000007748862)

  [1]: https://blog.screensteps.com/recovering-from-a-disastrous-git-rebase-mistake 
  [2]: https://stackoverflow.com/questions/161813/how-to-resolve-merge-conflicts-in-git