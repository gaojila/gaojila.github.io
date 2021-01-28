# git获取与原仓库同步更新


# 场景

github 上 fork 原项目，如何将本仓库更新到最新版本？

# 方法

- 配置当前 fork 仓库的原地址

```bash
git remote add upstream <原仓库github地址>
```

- 查看当前仓库的远程仓库地址和原仓库地址

```bash
git remote -v
```

![](https://raw.githubusercontent.com/gaojila/images/master/git%E8%8E%B7%E5%8F%96%E4%B8%8E%E5%8E%9F%E4%BB%93%E5%BA%93%E5%90%8C%E6%AD%A5%E6%9B%B4%E6%96%B0/2019-08-16-17-12-03.png)

- 获取原仓库的更新。使用 fetch 更新，fetch 后会被存储在一个本地分支 upstream/master 上

```bash
git fetch upstream
```

- 合并到本地分支。切换到 master 分支，合并 upstream/master 分支。

```bash
git merge upstream/master
```

- 这个时候使用 git log 就用看到原仓库的更新了。

```bash
git log
```

- 如果需要自己的 github 上的 fork 的仓库保持同步更新，执行 git push 进行推送。

```bash
git push origin master
```

- 如果出现无法提交时，强制覆盖。

```bash
git fetch origin
git reset --hard origin/master
```

