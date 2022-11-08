# Turborepo in Monorepos

## 问题所在 //The problem

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fwhy-turborepo-problem.863cb824.png&w=1920&q=75)

Monorepos有很多优点-但**它们在扩展方面有困难**。每个工作区都有自己的测试套件、自己的提示和自己的构建过程。一个单一的monorepo可能有**数百个任务要执行**。

## 解决办法 //The solution

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fwhy-turborepo-solution.a980f9ae.png&w=1920&q=75)

**Turborepo解决了你的monorepo的扩展问题**。我们的远程缓存存储了你所有任务的结果，这意味着**你的CI永远不需要做两次相同的工作**。

任务调度在单版本中可能是很困难的。想象一下，`yarn build`需要在`yarn test`之前运行，跨越所有的工作空间。Turborepo可以在所有可用的核心上，**以最大的速度调度你的任务**。

Turborepo 可以**渐进式采用**。它使用你已经写好的 `package.json` 脚本，你已经声明的依赖关系，以及一个单独的 `turbo.json` 文件。你可以**在任何软件包管理器中使用它**，如`npm`、`yarn`或`pnpm`。你可以在几分钟内把它添加到任何monorepo。

## turborepo 不是什么//What turborepo is not

Turborepo并**没有处理[软件包的安装](https://turbo.build/repo/docs/handbook/package-installation)**。像`npm`、`pnpm`或`yarn`这样的工具已经做得很出色了。但它们运行任务的效率很低，这意味着CI构建的速度很慢。

我们建议**Turborepo运行你的任务**，而你最喜欢的包管理器安装你的包