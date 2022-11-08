# 筛选工作空间 //Filtering Workspaces

一个monorepo可以包含成百上千的工作空间。默认情况下，运行`turbo run test`将在**所有可用的工作区**执行`test`任务。

![Without using a filter, test will be run across all packages](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fno-filter.54c37943.png&w=1920&q=75)

Turborepo支持一个`--filter`标志，让你**选择你想执行任务的工作空间**。

![With a filter on shared, only the shared package runs test](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fwith-filter.d1f5c746.png&w=1920&q=75)

你可以用它来:

- 按[工作区名称](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#filter-by-workspace-name)过滤
- 按[工作区目录](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#filter-by-directory)过滤
- 包括匹配的工作空间的[依赖](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#include-dependents-of-matched-workspaces)和[依附关系](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#include-dependencies-of-matched-workspaces)
- 从[工作区根目录](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#the-workspace-root)执行任务
- 根据[git历史中的变化](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#filter-by-changed-workspaces)进行过滤
- 从选择中[排除工作区](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#excluding-workspaces)

Turborepo将针对每个匹配的工作空间运行每个任务，确保根据[`turbo.json`](https://turbo.build/repo/docs/reference/configuration#pipeline)中的`管道`规范，首先运行任何依赖它的任务。

## 过滤器语法 //Filter Syntax

### 多个过滤器 //Multiple filters

你可以通过向命令传递多个`--filter`标志来指定一个以上的过滤器:

```sh
turbo run build --filter=my-pkg --filter=my-app
```

### 按工作区名称过滤 //Filter by workspace name

当你想只在一个工作区运行脚本时，你可以使用一个过滤器：`--filter=my-pkg`。

```sh
# Build 'my-pkg', letting `turbo` infer task dependencies
# from the pipeline defined in turbo.json
turbo run build --filter=my-pkg

# Build '@acme/bar', letting `turbo` infer task dependencies
# from the pipeline defined in turbo.json
turbo run build --filter=@acme/bar
```

如果你想在几个名字相似的工作空间内运行任务，你可以使用glob语法: `--filter=*my-pkg*`.

```sh
# Build all workspaces that start with 'admin-', letting turbo infer task
# dependencies from the pipeline defined in turbo.json
turbo run build --filter=admin-*
```

#### 范围

有些单选题在其工作空间名称前加上一个范围，如`@acme/ui`和`@acme/app`。只要这个范围（`@acme`）在整个代码库中是唯一的，你就可以从过滤器中省略它。

```diff
- turbo run build --filter=@acme/ui
+ turbo run build --filter=ui
```

### 包括匹配的工作空间的依赖 //Include dependents of matched workspaces

有时，你会想确保你的共享包不会影响任何下游的依赖。为此，你可以使用 `--filter=...my-lib`。

如果`my-app`依赖于`my-lib`， `...my-lib` 将选择`my-app`和`my-lib`。

包括一个`^` (`...^my-lib`)将选择所有`my-lib`的依赖，但不包括`my-lib`本身。

```sh
# Test 'my-lib' and everything that depends on 'my-lib'
turbo run test --filter=...my-lib

# Test everything that depends on 'my-lib', but not 'my-lib' itself
turbo run test --filter=...^my-lib
```

### 包括匹配的工作空间的依赖关系 //Include dependencies of matched workspaces

有时，你会想确保在你所针对的lib的所有依赖中运行`build`。为此，你可以使用 `--filter=my-app...`。

If `my-app` depends on `my-lib`, `my-app...` will select `my-app` and `my-lib`.
如果`my-app`依赖于`my-lib`，`my-app...`将选择`my-app` 和 `my-lib`。

包括`^`（`my-app^...`）将选择`my-app`的所有依赖，但不包括`my-app`本身。

```sh
# Build 'my-app' and its dependencies
turbo run build --filter=my-app...

# Build 'my-app's dependencies, but not 'my-app' itself
turbo run build --filter=my-app^...
```

### 按目录过滤 //Filter by directory

当你想针对一个特定的目录，而不是一个工作区名称时，这很有用。它支持。

- 完全匹配: `--filter=./apps/docs`
- 全局: `--filter=./apps/*`

```sh
# Build all of the workspaces in the 'apps' directory
turbo run build --filter=./apps/*
```

#### 与其他语法结合 //Combining with other syntaxes

当把目录过滤器与其他语法结合起来时，用`{}`括起来。例如:

```sh
# Build all of the workspaces in the 'libs' directory,
# and all the workspaces that depends on them
turbo run build --filter=...{./libs/*}
```

### 按改变的工作空间过滤 //Filter by changed workspaces

你可以在某次提交后发生变化的任何工作空间上运行任务。这些任务需要用`[]`来包装。

例如，`--filter=[HEAD^1]`将选择所有在最近一次提交中发生变化的工作区:

```sh
# Test everything that changed in the last commit
turbo run test --filter=[HEAD^1]
```

#### 检查一系列的提交 //Check a range of commits

如果您需要检查特定范围的提交，而不是与`HEAD`比较，您可以通过`[<from commit>...<to commit>]`设置比较的两端

```sh
# Test each workspace that changed between 'main' and 'my-feature'
turbo run test --filter=[main...my-feature]
```

#### 忽略已更改的文件 //Ignoring changed files

你可以使用[`--ignore`](https://turbo.build/repo/docs/reference/command-line-reference#--ignore)来指定在计算哪些工作空间发生了变化时要忽略的变化的文件。

#### 与其他语法的结合 //Combining with other syntaxes

你还可以在提交引用前加上`...`，以匹配其他组件与变更后的工作区的依赖关系。例如，如果要选择`foo`的任何依赖关系在上次提交中发生了变化，可以通过`--filter=foo...[HEAD^1]`。

```sh
# Build everything that depends on changes in branch 'my-feature'
turbo run build --filter=...[origin/my-feature]

# Build '@foo/bar' if it or any of its dependencies
# changed in the last commit
turbo run build --filter=@foo/bar...[HEAD^1]
```

你甚至可以把`[]`和{}的语法结合在一起:

```sh
# Test each workspace in the '@scope' scope that
# is in the 'packages' directory, if it has
# changed in the last commit
turbo run test --filter=@scope/*{./packages/*}[HEAD^1]
```

### 工作区的根目录 //The workspace root

monorepo's的根目录可以用标记`//`选择

```sh
# Run the format script from the root "package.json" file:
turbo run format --filter=//
```

### 排除工作空间 //Excluding workspaces

在过滤器前加上 `!`。整个过滤器中匹配的工作区将从目标集合中排除。例如，匹配除 `@foo/bar`: `--filter=!@foo/bar`.之外的所有内容。注意，你可能需要根据你的shell来转义`!`（例如：`\!`！）。

```sh
# Build everything except '@foo/bar'
turbo run build --filter=!@foo/bar
# Build all of the workspaces in the 'apps' directory, except the 'admin' workspace
turbo run build --filter=./apps/* --filter=!admin
```

Turborepo的Filter API设计和文档是/是受到了[pnpm](https://pnpm.io/filtering)的启发