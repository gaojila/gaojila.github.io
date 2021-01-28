# gitrest与gitrebase


# 场景

有时候已经提交到远程仓库的代码希望回滚该怎么做？
有时候有多个 commit 该如何合并成一个提交？

# 方法

- 使用 git rebase 合并多个 commit

```bash
git rebase -l <版本号>
```

- 使用 git rebase 回滚已经提交

```bash
git reset --soft <版本号>
git push origin master --force
```

# 注意

- 需要在 web 端关闭分支保护，然后重新 git add

