# 配置选项 //Configuration Options (`turbo.json`)

你可以通过在你的monorepo的根目录下添加一个`turbo.json`文件来配置`turbo`的行为（即你指定你的`workspaces`密钥为Yarn和npm用户设置的同一个文件）。

## `globalDependencies`

`type: string[]`

A list of file globs for implicit global hash dependencies. The contents of these files will be included in the global hashing algorithm and affect the hashes of all tasks.
This is useful for busting the cache based on `.env` files (not in Git) or any root level file that impacts workspace tasks (but are not represented in the traditional dependency graph (e.g. a root `tsconfig.json`, `jest.config.js`, `.eslintrc`, etc.)).
隐式全局散列依赖的文件globs列表。这些文件的内容将被包含在全局散列算法中，并影响所有任务的散列。这对于基于`.env`文件（不在Git中）或任何影响工作区任务的根级文件（但不在传统的依赖关系图中表示（例如根目录`tsconfig.json`、`jest.config.js`、`.eslintrc`等）的破坏缓存很有用。）

**列子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    // ... omitted for brevity
  },

  "globalDependencies": [
    ".env", // contents will impact hashes of all tasks
    "tsconfig.json" // contents will impact hashes of all tasks
  ]
}
```

## `globalEnv`

`type: string[]`

隐含全局哈希依赖的环境变量列表。这些环境变量的内容将包括在全局散列算法中，并影响所有任务的散列。

**例子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    // ... omitted for brevity
  },

  "globalEnv": ["GITHUB_TOKEN"] // value will impact the hashes of all tasks
}
```

## `pipeline`

一个代表你的项目的任务依赖关系图的对象。`turbo`解释这些约定，以正确安排、执行和缓存你的项目中的任务的输出。

`管道`对象中的每个键都是可以被`turbo run`执行的任务的名称。如果`turbo`发现一个工作区的`package.json` `scripts`对象有一个匹配的键，它将在执行过程中把管道任务配置应用到该npm脚本。这允许你在整个Turborepo中使用`管道`来设置惯例。

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"]
    },
    "test": {
      "outputs": ["coverage/**"],
      "dependsOn": ["build"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"],
      "outputMode": "full"
    },
    "dev": {
      "cache": false
    }
  }
}
```

### `dependsOn`

`type: string[]`

这个任务所依赖的任务列表。

在`dependOn`中的项目前缀为`^`，告诉`turbo`这个管道任务取决于工作空间的拓扑依赖项是否先完成了`^`前缀的任务（例如，"一个工作空间的`build`任务应该只在其所有的`dependencies`和`devDependencies`完成自己的`build`命令后运行"）。

`dependsOn`中的项目没有`^`前缀，表达了工作区级别的任务之间的关系（例如，"一个工作区的`test`和`lint`命令取决于首先完成的`build`"）。

在 `dependsOn` 中的一个项目前加上 `$`，告诉 `turbo` 这个管道任务依赖于该环境变量的值。

在 `dependsOn` 配置中使用 `$` 来声明环境变量的做法已被弃用。请使用 `env` 键来代替。

**例子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    "build": {
      // "A workspace's `build` command depends on its dependencies'
      // or devDependencies' `build` command being completed first"
      "dependsOn": ["^build"]
    },
    "test": {
      // "A workspace's `test` command depends on its own `lint` and
      // `build` commands first being completed"
      "dependsOn": ["lint", "build"]
    },
    "deploy": {
      // "A workspace's `deploy` command, depends on its own `build`
      // and `test` commands first being completed"
      "dependsOn": ["build", "test"]
    },
    // A workspace's `lint` command has no dependencies
    "lint": {}
  }
}
```

### `env`

`type: string[]`

一个任务所依赖的环境变量列表。

**例子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "env": ["SOMETHING_ELSE"], // value will impact the hashes of all build tasks
      "outputs": ["dist/**", ".next/**"]
    },
    "web#build": {
      "dependsOn": ["^build"],
      "env": ["STRIPE_SECRET_KEY"], // value will impact hash of only web's build task
      "outputs": [".next/**"]
    }
  },
  "globalEnv": [
    "GITHUB_TOKEN" // value will impact the hashes of all tasks
  ]
}
```

当Turborepo检测到工作空间中的通用前端框架时，它将自动依赖环境变量，这些环境变量将在你的构建中被内联。例如，如果 `web` 工作区包含 Next.js 项目，你不需要在 `dependsOn` 配置中指定任何以 [`NEXT_PUBLIC_` 开始](https://nextjs.org/docs/basic-features/environment-variables#exposing-environment-variables-to-the-browser)的环境变量。Turborepo 已经知道，当这些环境变量的值发生变化时，构建输出会发生变化，所以它会自动依赖这些变量。请看更多关于[缓存的文档](https://turbo.build/repo/docs/core-concepts/caching#automatic-environment-variable-inclusion)。

### `outputs`

`type: string[]`

默认为`["dist/**", "build/**"]`。一个任务的可缓存文件系统输出的glob模式集合。

注意：`turbo`自动将`stderr`/`stdout`记录到`.turbo/run-<task>.log`。这个文件 _总是_ 被视为一个可缓存的工件，永远不需要被指定。

传递一个空数组可以用来告诉`turbo`，一个任务是一个副作用，因此不会发出任何文件系统工件（例如像linter），但你仍然想缓存它的日志（并把它们当作一个工件）。

**例子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    "build": {
      // "Cache all files emitted to workspace's dist/** or .next
      // directories by a `build` task"
      "outputs": ["dist/**", ".next/**"],
      "dependsOn": ["^build"]
    },
    "test": {
      // "Don't cache any artifacts of `test` tasks (aside from
      // logs)"
      "outputs": [],
      "dependsOn": ["build"]
    },
    "test:ci": {
      // "Cache the coverage report of a `test:ci` command"
      "outputs": ["coverage/**"],
      "dependsOn": ["build"]
    },
    "dev": {
      // Never cache anything (including logs) emitted by a
      // `dev` task
      "cache": false
    }
  }
}
```

### `cache`

`type: boolean`

默认为`true`。是否对任务[`输出`](https://turbo.build/repo/docs/reference/configuration#outputs)进行缓存。将`cache`设置为false对于daemon或长期运行的 "watch "或开发模式的任务是有用的，你不希望缓存。

**例子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"]
    },
    "test": {
      "outputs": [],
      "dependsOn": ["build"]
    },
    "dev": {
      "cache": false
    }
  }
}
```

### `inputs`

`type: string[]`

默认为`[]`。告诉`turbo`在确定某项任务的工作区是否有变化时要考虑的文件集。将此设置为globs列表将导致任务只在与这些globs匹配的文件发生变化时才被重新运行。如果你想，例如，跳过运行测试，除非一个源文件改变了，这可能是有帮助的。

指定`[]`将导致任务在工作区的任何文件发生变化时被重新运行。

**例子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    // ... omitted for brevity

    "test": {
      // A workspace's `test` task depends on that workspace's
      // own `build` task being completed first.
      "dependsOn": ["build"],
      "outputs": [""],
      // A workspace's `test` task should only be rerun when
      // either a `.tsx` or `.ts` file has changed.
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"]
    }
  }
}
```

注意：`turbo.json`*总是*被认为是一个输入。如果你修改了`turbo.json`，所有的缓存都会被废止。

### `outputMode`

`type: "full" | "hash-only" | "new-only" | "none"`

设置输出记录的类型。

|  option   | description  |
|  ----  | ----  |
| full  | 这是默认的。显示所有输出 |
| hash-only  | 只显示任务的哈希值 |
| new-only  | 只显示缓存缺失的输出 |
| none  | 隐藏所有的任务输出 |

**例子**

```jsonc
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputMode": "new-only"
    },
    "test": {
      "outputs": [],
      "dependsOn": ["build"]
    }
  }
}
```