---
title: "Git"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

git reset

git reverse



git rebase



git rebase main

当前在bugFix这个branch，把当前的branch copy到main branch



git rebase a b

将b复制到a下



git rebase bugFix



git rebase bugFix main



git pull --rebase



git push origin main (git push <remote> <place>)

git push origin <source本地>:<destination远>

如果<source>是空的，例如 git push origin :foo ，那就是删除foo on remote



git fetch 和 git pull的区别？git pull = git fetch + merge

git fetch origin <source远>:<destination本地>

如果<source>是空的，例如git fetch origin :bar, 那就是新建bar 分支