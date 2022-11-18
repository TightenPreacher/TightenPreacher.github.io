
# 在Monorepo中对软件包进行版本管理和发布 //Versioning and Publishing Packages in a Monorepo

在monorepo中手动版本管理和发布软件包是非常累人的。幸运的是，有一个工具可以让事情变得简单-[Changesets](https://github.com/changesets/changesets) CLI。

我们推荐Changesets，因为它使用起来很直观，而且-就像Turborepo一样-适合你已经习惯的monorepo工具。

一些替代品是:

- [intuit/auto](https://github.com/intuit/auto) - 根据拉动请求上的语义版本标签来生成版本
- [microsoft/beachball](https://github.com/microsoft/beachball) - 最阳光的语义版本缓冲器

## 了解Changesets //Understanding Changesets

我们建议看一下Changesets的文档。以下是我们推荐的阅读顺序:

1. [为什么使用 changesets?](https://github.com/changesets/changesets/blob/main/docs/intro-to-using-changesets.md) - 一个介绍，带你了解基本原理。
2. [安装说明](https://github.com/changesets/changesets/blob/main/packages/cli/README.md)
3. 如果你使用的是GitHub，可以考虑使用[Changeset GitHub机器人](https://github.com/apps/changeset-bot)-一个可以催促你向PR添加变更集的机器人。
4. 你也应该考虑添加[Changesets GitHub动作](https://github.com/changesets/action) - 一个让发布变得非常简单的工具。

## 使用Turborepo的Changesets //Using Changesets with Turborepo

一旦你开始使用 Changesets，你将获得三个有用的命令:

```bash
# Add a new changeset
changeset

# Create new versions of packages
changeset version

# Publish all changed packages to npm
changeset publish
```

将您的发布流程链接到Turborepo，可以使您的部署组织工作变得更加简单和快速。

我们的建议是在你的根`package.json`中添加一个`publish-packages`脚本:

```json
filename="package.json"
{
  "scripts": {
    // Include build, lint, test - all the things you need to run
    // before publishing
    "publish-packages": "turbo run build lint test && changeset version && changeset publish"
  }
}
```

我们推荐`publish-package`s，这样它就不会与npm内置的`publish`脚本冲突。

这意味着当你运行 `publish-packages` 时，你的 monorepo 会被构建、加注、测试和发布 - 你会受益于 Turborepo 的所有加速。