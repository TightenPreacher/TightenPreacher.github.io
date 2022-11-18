# 什么是 Monorepo?

monorepo 是一个单一代码库中许多不同应用程序和包的集合。

另一种设置被称为**polyrepo**-多个代码库被单独发布和版本管理。

## 共享代码

### 在 polyrepo 中

在polyrepo设置中，应用程序之间共享代码的过程相对较长。

想象一下，你有三个独立的仓库-`app`、`docs`和`share-utils`。`app`和`docs`都依赖于`share-utils`，而share-utils是作为一个包发布在npm上的。

假设 `shared-utils` 中的一个 bug 导致了 `app` 和 `docs` 的一个关键问题。你需要:

1. 在`share-utils`中提交一个修复错误的文件
2. 在`share-utils`中运行一个`发布`任务，将其发布到npm中
3. 在`app`中提交，增加`share-utils`的依赖版本
4. 在`docs`中提交，增加`share-utils`的依赖版本
5. `app`和`docs`现在可以部署了

你有越多的应用程序依赖`share-utils`，这个过程就越长。这可能是非常艰巨的。

### 在 monorepo 中

在monorepo设置中，`shared-utils`将与`app`和`docs`在 _同一个代码库_ 中。这使得这个过程非常简单。

1. 在 `shared-utils` 中提交一个修正错误的文件
2. `app` 和 `docs` 现在就可以部署了。

不需要版本管理，因为`app`和`docs`并不依赖于npm中`share-utils`的版本，它们依赖于**代码库中的版本**。

这使得创建单一的提交可以同时修复多个应用程序和软件包中的错误。这对团队来说是一个巨大的速度提升。

## monorepos怎么工作的?

monorepo的主要构建模块是[workspace](https://turbo.build/repo/docs/handbook/workspaces)。你构建的每个应用程序和包都将在它自己的工作空间中，有它自己的`package.json`。正如你将从我们的指南中了解到的，工作区可以**相互依赖**，这意味着你的`docs`工作区可以依赖`share-utils`:


### npm
```json 
filename="apps/docs/package.json"
{
  "dependencies": {
    "shared-utils": "*"
  }
}
```

### yarn
```json 
filename="apps/docs/package.json"
{
  "dependencies": {
    "shared-utils": "*"
  }
}
```

### pnpm
```json 
filename="apps/docs/package.json"
{
  "dependencies": {
    "shared-utils": "workspace:*"
  }
}
```

工作空间是由[安装你的依赖关系](https://turbo.build/repo/docs/handbook/package-installation)的同一个CLI管理的。

### 根工作区 //The root workspace

你也会有一个根工作空间-在你的代码库的根文件夹中的`package.json`。这是一个有用的地方:

1. 指定存在于整个monorepo中的依赖关系
2. 添加对 _整个_ monorepo进行操作的任务，而不仅仅是单个工作区
3. 添加关于如何使用monorepo的文档