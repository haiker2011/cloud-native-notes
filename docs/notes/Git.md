# 使用 Github 贡献代码最佳实践

## 原则1

 > 远程分支与 upstream 分支始终保持一致，在 master 新建一个 dev 分支进行开发，开发完成之后发起 PR,若有冲突，更新 master，然后再 dev 分支解决冲突，PR 处理完成之后删掉 dev 分支，下次新的 PR 之前先与 upstream 保持最新，然后建立新分支 dev。

## 原则2

> 若基于 upstream 的非主干 master 分支开发，现在本地通过 `git checkout -b release-1.0` 新建一个同名的分支，然后在该分支上创建 dev 分支进行开发，同上。

## 原则3

> 若需要删除远程分支上的一个分支，例如 `release-1.0`，可以使用 `git push origin --delete release-1.0` 进行删除。


## 参考文献

1. [删除远程分支](https://www.cnblogs.com/luosongchao/p/3408365.html)
2. [同步非master分支](https://segmentfault.com/q/1010000004228020/a-1020000004228167)
3. [保持仓库同步](https://blog.csdn.net/csm201314/article/details/83045605)
4. [github中fork分支和pullrequest的最佳实践](https://www.cnblogs.com/yangwen0228/p/6747483.html?utm_source=itdadao&utm_medium=referral)