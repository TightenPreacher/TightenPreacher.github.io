
# 在Monorepo中捆绑软件包

与[内部](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages)包不同，外部包可以被部署到npm _并在_ 本地使用。在本指南中，我们将使用捆绑器将一个包捆绑到[`CommonJS`](https://en.wikipedia.org/wiki/CommonJS)，这是npm上最常用的格式。

## 设置一个构建脚本 //Setting up a build script

让我们从一个使用[内部包](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages)教程创建的包开始。

在那里，我们创建了一个`math-helpers`包，其中包含一些加减法的辅助函数。我们已经决定这个包对npm来说足够好了，所以我们要把它捆绑起来。

在那个教程的最后，我们在`/packages`下设置了一个包，它看起来像这样:

```
├── apps
│   └── web
│       └── package.json
├── packages
│   └── math-helpers
│       ├── src
│       │   └── index.ts
│       ├── tsconfig.json
│       └── package.json
├── package.json
└── turbo.json
```

我们将使用一个捆绑器为 `math-helpers` 添加一个`构建`脚本。如果你不确定该选择哪一个，我们推荐[`tsup`](https://tsup.egoist.dev/)。

首先，使用你的[软件包管理器](https://turbo.build/repo/docs/handbook/package-installation)在`packages/math-helpers`里面安装`tsup`。

```json 
filename="packages/math-helpers/package.json"
{
  "scripts": {
    "build": "tsup src/index.ts --format cjs --dts"
  }
}
```

`tsup`默认输出文件到`dist`目录，所以你应该这样做:

1. 在你的`.gitignore`文件中加入`dist`，以确保它们不被`git`提交。
2. 在你的`turbo.json`中把`dist`加到`build`的输出中。

```json 
filename="turbo.json"
{
  "pipeline": {
    "build": {
      "outputs": ["dist/**"]
    }
  }
}
```

这样，当`tsup`运行时，输出可以被Turborepo[缓存](https://turbo.build/repo/docs/core-concepts/caching)。

最后，我们应该把`main`改为指向`package.json`里面的`./dist/index.js`。类型可以指向`./dist/index.d.ts`

```json 
filename="packages/math-helpers/package.json"
{
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts"
}
```

如果你通过使用`main`和`types`遇到了错误，请看一下[tsup文档](https://tsup.egoist.dev/#bundle-formats)。

捆绑是一个复杂的话题，我们在这里没有足够的空间来涵盖所有的内容。

### 在我们的应用程序之前构建我们的包 //Building our package before our app

在我们运行`Turbo run build`之前，有一件事我们需要考虑。我们刚刚在我们的monorepo中添加了一个[任务依赖项](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)。`packages/math-helpers`的`构建`需要在`apps/web`的构建**之前**运行。

幸运的是，我们可以使用 `dependsOn` 来轻松配置。

```json 
filename="turbo.json"
{
  "pipeline": {
    "build": {
      "dependsOn": [
        // Run builds in workspaces I depend on first
        "^build"
      ]
    }
  }
}
```

Now, we can run `turbo run build`, and it'll automatically build our packages _before_ it builds our app.
现在，我们可以运行`turbo run build`，它将在构建我们的应用程序 _之前_ 自动构建我们的软件包。

## 设置一个开发脚本 //Setting up a dev script

我们的设置有一个小问题。我们正在构建我们的软件包，但在开发中却不太顺利。我们对 `math-helpers` 包所做的修改并没有反映在我们的应用程序中。

这是因为我们没有一个`dev`脚本来在我们工作时重建我们的包。我们可以很容易地添加一个:

```json 
filename="packages/math-helpers/package.json"
{
  "scripts": {
    "build": "tsup src/index.ts --format cjs --dts",
    "dev": "npm run build --watch"
  }
}
```

这将传递给`tsup` `--watch`标志，意味着它将监视文件的变化。

如果我们已经在`turbo.json`中设置了[开发脚本](https://turbo.build/repo/docs/handbook/dev)，运行`turbo run dev`就会在`apps/web`开发任务中并行运行`packages/math`开发任务。

## 总结

我们的软件包现在已经到了可以考虑部署到npm的位置。在我们的[版本管理和发布](https://turbo.build/repo/docs/handbook/publishing-packages/versioning-and-publishing)部分，我们将做到这一点。
