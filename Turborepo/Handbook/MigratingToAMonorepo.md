
# 迁移到Monorepo //Migrating to a Monorepo

从多版本设置迁移到单版本设置可以为生产力带来巨大的好处，特别是在以下情况:

- 你发现应用程序之间很难共享代码
- 你希望你的团队有一个统一的方法来构建代码

迁移到monorepo可能是令人生畏的。但只要仔细规划，就可以很顺利地进行。

## 文件夹结构 //Folder structure

让我们想象一下，你的多版本设置是这样的:

```
web (repo 1)
├─ package.json

docs (repo 2)
├─ package.json

app (repo 3)
├─ package.json
```

你有三个资源库， `web`, `docs`和`app`。它们没有任何共享的依赖关系，但你注意到它们之间有很多 _重复_ 的代码。

将它们组织在一个单库中的最好方法是这样的:

```
my-monorepo
├─ apps
│  ├─ app
│  │  └─ package.json
│  ├─ docs
│  │  └─ package.json
│  └─ web
│     └─ package.json
└─ package.json
```

为了开始分享代码，你可以使用[内部包模式](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages)，从而形成一个新的`packages`文件夹

```
my-monorepo
├─ apps
│  ├─ app
│  │  └─ package.json
│  ├─ docs
│  │  └─ package.json
│  └─ web
│     └─ package.json
├─ packages
│  └─ shared
│     └─ package.json
└─ package.json
```

如果你打算转为单机版，试着勾勒出你所追求的确切的文件夹结构。

## 设置工作空间 //Setting up workspaces

一旦你的应用程序在正确的文件夹结构中，你将需要设置工作空间并安装你的依赖关系。我们关于[设置工作空间](https://turbo.build/repo/docs/handbook/workspaces)的章节应该有所帮助。

## 处理任务 //Handling tasks

现在你的工作空间已经设置好了，你需要弄清楚你将如何在新的monorepo中运行你的任务。我们有以下章节:

1. 如何用Turborepo[配置任务](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)
2. 如何设置你的[开发任务](https://turbo.build/repo/docs/handbook/dev)
3. 如何设置[linting](https://turbo.build/repo/docs/handbook/linting)
4. 如何[构建你的应用程序](https://turbo.build/repo/docs/handbook/building-your-app)
5. 如何设置[testing](https://turbo.build/repo/docs/handbook/testing)
