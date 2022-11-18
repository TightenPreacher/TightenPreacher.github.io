# 在monorepo上Linting //Linting in a monorepo

在monorepo中Linting是很棘手的。你的大部分工作区都可能包含需要加注的代码-所以找出最有效的方法来加注它们是很困难的

在本指南中，我们将提出一个发挥Turborepo优势的方法

- 在工作区 _中_ 运行lint任务，而不是从根目录中刷新
- 在工作区之间尽可能多的共享配置

## 运行任务 //Running tasks

我们建议在你的`turbo.json`中指定一个单一的`lint`任务

```json 
filename="turbo.json"
{
  "pipeline": {
    "lint": {
      "outputs": []
    }
  }
}
```

然后，在**每个需要提示的工作区里面**，添加一个`lint`脚本。我们将使用TypeScript作为一个例子

```json 
filename="packages/*/package.json"
{
  "scripts": {
    "lint": "tsc"
  }
}
```

这种模式有两个好处:

- [**并行化**](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks): lint任务将被并发运行，加快了它们的速度
- [**缓存**](https://turbo.build/repo/docs/core-concepts/caching): `lint`任务将只在发生变化的工作区重新运行。

这意味着你可以用一个命令对你的整个 repo 进行lint:

```bash
turbo run lint
```

## 共享配置文件 //Sharing config files

在一个单子上共享配置有助于保持开发经验的一致性。大多数linters会有一个系统来共享配置，或在不同的文件中扩展配置。

到目前为止，我们已经建立了共享配置文件的指南，在:

- [TypeScript](https://turbo.build/repo/docs/handbook/linting/typescript)
- [ESLint](https://turbo.build/repo/docs/handbook/linting/eslint)