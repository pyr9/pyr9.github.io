---
title: git修改指定的提交
date: 2022-05-10 21:57:27
tags: Git
categories: Git
---

# 1.  git修改指定的提交

- `git rebase --interactive 'cffa46f19811496d9165cf2c32250943f94f0099^'`
- 修改
- `git add .`
- `git commit --amend`
- `git rebase --continue`

# 2. 更新gitignore文件后，git删除仓库中的缓存

`git rm -r --cached target/ `
