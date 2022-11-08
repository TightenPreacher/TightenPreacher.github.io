
# 在Monorepo中运行任务 //Running Tasks in a Monorepo

每个monorepo都有两个主要的构建模块：**工作空间**和**任务**。让我们想象一下，你有一个包含**三个工作空间**的单例，每个工作空间**有三个任务**

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fyour-monorepo-excalidraw.ccf1c6c1.png&w=1920&q=75)

在这里，`apps/web`和`apps/doc`都使用`packages/shared`的代码。事实上，当它们被构建时（通过`build`），**它们需要 _首先_ 构建 `packages/shared`**。

## 大多数工具都没有对速度进行优化 //Most tools don't optimize for speed

让我们想象一下，我们想在所有的工作空间中运行所有的任务。在`yarn`这样的工具中，你可以运行这样的脚本:

```
yarn workspaces run lint
yarn workspaces run test
yarn workspaces run build
```

这将意味着任务的运行方式是这样的:

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fyarn-workspaces-excalidraw.0838365d.png&w=1920&q=75)

正如你所看到的，在所有的工作区都会运行`lint`。然后，`build`被运行-首先是`shared`。最后，`test`被运行。

**这是运行这些任务的最慢的方式**。每个任务都需要等待前一个任务完成后才能开始。为了改进这一点，我们需要一个可以多任务的工具。

## Turborepo可以进行多任务处理 //Turborepo can multitask

Turborepo 可以通过了解任务之间的依赖关系来安排我们的任务，以达到最大的速度。

首先，我们在 `turbo.json` 中声明我们的任务:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      // ^build means build must be run in dependencies
      // before it can be run in this workspace
      "dependsOn": ["^build"]
    },
    "test": {
      "outputs": []
    },
    "lint": {
      "outputs": []
    }
  }
}
```

接下来，我们可以用这个来替换我们的`yarn workspaces`脚本:

```diff
- yarn workspaces run lint
- yarn workspaces run test
- yarn workspaces run build
+ turbo run lint test build
```

当我们运行它时，Turborepo会在**所有可用的CPU上尽可能多地执行任务**，这意味着我们的任务会像这样运行

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fturborepo-excalidraw.8068f4b4.png&w=1920&q=75)

`lint`和`test`都立即运行，因为它们在`turbo.json`中没有指定`dependOn`

`shared`里面的`构建`任务首先完成，然后是`web`和`docs`的构建。

## 定义一个流水线 //Defining a pipeline

`管线`配置声明了在你的 monorepo 中哪些任务是相互依赖的。这里有一个厨房水槽的例子:

```diff
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
+      // A workspace's `build` task depends on that workspace's
+      // topological dependencies' and devDependencies'
+      // `build` tasks  being completed first. The `^` symbol
+      // indicates an upstream dependency.
      "dependsOn": ["^build"]
    },
    "test": {
+      // A workspace's `test` task depends on that workspace's
+      // own `build` task being completed first.
      "dependsOn": ["build"],
      "outputs": [],
+      // A workspace's `test` task should only be rerun when
+      // either a `.tsx` or `.ts` file has changed.
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts", "test/**/*.tsx"]
    },
    "lint": {
+      // A workspace's `lint` task has no dependencies and
+      // can be run whenever.
      "outputs": []
    },
    "deploy": {
+      // A workspace's `deploy` task depends on the `build`,
+      // `test`, and `lint` tasks of the same workspace
+      // being completed.
      "dependsOn": ["build", "test", "lint"],
      "outputs": []
    }
  }
}
```

让我们在深入研究`turbo.json`之前，先了解一下一些常见的模式。

### 任务之间的依赖性 //Dependencies between tasks

#### 在同一工作区在同一工作区 //In the same workspace

可能有一些任务需要在其他任务 _之前_ 运行。例如，在`deploy`前可能需要运行`build`。

如果这两个任务都在同一个工作区，你可以像这样指定关系:

```json filename="turbo.json"
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {},
    "deploy": {
      // A workspace's `deploy` task depends on the `build`,
      // task of the same workspace being completed.
      "dependsOn": ["build"],
      "outputs": []
    }
  }
}
```

这意味着，每当运行`turbo run deploy`时，`build`也将在同一工作空间内运行。

#### 在一个不同的工作空间 //In a different workspace

monorepos中的一个常见模式是声明一个工作区的`构建`任务只有在 _它所依赖的所有工作区_ 的`构建`任务完成后才能运行。

`^`符号明确声明该任务与它所依赖的工作空间中的任务有依赖关系。

```jsonc 
filename="turbo.json"
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      // "A workspace's `build` command depends on its dependencies'
      // and devDependencies' `build` commands being completed first"
      "dependsOn": ["^build"]
    }
  }
}
```

#### 无依赖性 //No dependencies

一个空的依赖列表（`dependencyOn`为未定义或`[]`）意味着在这个任务之前不需要运行任何东西。毕竟，它没有依赖性。

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "lint": {
      // A workspace's `lint` command has no dependencies and can be run
      // whenever.
      "outputs": []
    }
  }
}
```

#### 特定的工作区任务 //Specific workspace-tasks

有时，你可能想在另一个工作区任务上创建一个工作区任务的依赖关系。这对于从 `lerna` 或 `rush` 迁移的 repos 特别有帮助，在这些地方，任务默认是在不同的阶段运行。有时，这些配置做出的假设无法在简单的`管道`配置中表达，如上图所示。或者你可能只是想在CI/CD中使用`turbo`时表达应用程序或微服务之间的任务序列。

对于这些情况，你可以使用`<workspace>#<task>`语法在你的`管道`配置中表达这些关系。下面的例子描述了一个`前端`应用程序的`部署`脚本，它依赖于`后端`的`deploy` 和 `health-check`脚本，以及一个`ui`工作区的`test`脚本。

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    // Standard configuration
    "build": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "deploy": {
      "dependsOn": ["test", "build"],
      "outputs": []
    },

    // Explicit workspace-task to workspace-task dependency
    "frontend#deploy": {
      "dependsOn": ["ui#test", "backend#deploy", "backend#health-check"],
      "outputs": []
    }
  }
}
```

`frontend#deploy`的这种明确配置似乎与`test` 和 `deploy`任务的配置相冲突，但其实不然。由于`test`和`deploy`并不依赖其他工作空间（例如`^<task>`），它们可以在其工作空间的`build` 和 `deploy`脚本完成后的任何时间执行。

  注意:

1. 尽管这种 `<workspace>#<task>` 语法是一种有用的逃逸方式，但我们通常建议将其用于部署协调任务，如健康检查，而不是构建时的依赖，这样 Turborepo 可以更有效地优化这些任务
2. 打包任务不继承缓存配置。你必须在此刻重新声明[输出](https://turbo.build/repo/docs/reference/configuration#outputs)
3. `<workspace>`必须与工作区`package.json`中的`name`键匹配，否则任务将被忽略。


### 从根目录运行任务 //Running tasks from the root

`turbo`可以运行存在于monorepo根目录的`package.json`文件中的任务。这些任务必须使用关键语法`"//#<task>"`明确添加到管道配置中。即使是那些已经有自己条目的任务也是如此。例如，如果你的管道声明了一个 `"build"`任务，而你想把定义在monorepo的根package.json文件中的`build`脚本与`turbo run build`一起包含进去，你必须通过声明`"//#build": {...}`把根选择进去。。反之，你不需要定义一个通用的 `"my-task": {...}`条目，如果你只需要`"//#my-task": {...}`.

一个定义了根任务`格式`并将根任务选择为`test`的管道样本可能看起来像。

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    // This will cause the "test" script to be included when
    // "turbo run test" is run
    "//#test": {
      "dependsOn": [],
      "outputs": []
    },
    // This will cause the "format" script in the root package.json
    // to be run when "turbo run format" is run. Since the general
    // "format" task is not defined, only the root's "format" script
    // will be run.
    "//#format": {
      "dependsOn": [],
      "outputs": ["dist/**/*"],
      "inputs": ["version.txt"]
    }
  }
}
```

**关于递归的说明**。在monorepo的根目录`package.json`中定义的脚本经常调用`turbo`本身。例如， `build`脚本可能是`turbo run build`。在这种情况下，包括`//#build`在内的`turbo run build`会导致无限的递归。正是由于这个原因，从monorepo根目录下运行的任务必须通过在管道配置中包括`//#<task>`来明确选择。`turbo`包括一些尽力的检查，以便在递归情况下产生错误，但这取决于你是否只选择那些本身不会触发会递归的`turbo`运行的任务。

Turborepo的Pipeline API设计和这页文档的灵感来自[微软的Lage项目](https://microsoft.github.io/lage/docs/Tutorial/pipeline#defining-a-pipeline)。向[Kenneth Chau](https://twitter.com/kenneth_chau)致敬，因为他提出了用这种简洁而优雅的方式来分担任务的想法。

### 提示

#### 在`管道`中但不在某些`package.json`中的任务 //Tasks that are in the `pipeline` but not in SOME `package.json`

有时，在`管道`中声明的任务并不存在于所有工作空间的`package.json`文件中。`turbo`将优雅地忽略这些。没问题!

#### `管道`任务是`Turbo`唯一知道的。 //`pipeline` tasks are the only ones that `turbo` knows about

`turbo`只会考虑到`管道`配置中声明的任务。如果它没有列在那里，`turbo`将不知道如何运行它们。
