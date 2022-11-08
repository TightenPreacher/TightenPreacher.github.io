# 缓存任务 //Caching Tasks

每个 JavaScript 或 TypeScript 代码库都需要运行 `package.json` 脚本，比如 `build`、`test` 和 `lint`。在Turborepo中，我们把这些称为**任务**

Turborepo 可以缓存任务的结果和日志 - 导致慢速任务的巨大速度提升。

## 缺少缓存 //Missing the cache

你代码库中的每个任务都有**输入**和**输出**。

- 一个`构建`任务可能有源文件作为输入，并将日志输出到 `stderr` 和 `stdout` 以及捆绑文件。
- 一个`lint`或`test`任务可能有源文件作为输入，并将日志输出到`stdout`和`stderr`。

假设你用Turborepo运行一个`build`任务，使用`turbo run build`。

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fcache-miss.21d45e92.png&w=1920&q=75)

1. Turborepo将**评估你的任务的输入**（默认为工作区文件夹中所有非gitignored的文件），并将**它们变成一个哈希值**（例如`78awdk123`）。

2. **检查本地文件系统的缓存**中是否有以该哈希值命名的文件夹（例如`./node_modules/.cache/turbo/78awdk123`）。

3. 如果Turborepo没有找到任何与计算出的哈希值相匹配的工件，Turborepo就会**执行这个任务**。

4. 一旦任务结束，Turborepo 会将**所有的输出（包括文件和日志）保存**到哈希值下的缓存中。

Turborepo在创建哈希值时考虑了很多信息-源文件、环境变量，甚至是依赖工作区的源文件。在[下面](https://turbo.build/repo/docs/core-concepts/caching#hashing)了解更多。

## 点击缓存 //Hitting the cache

假设你在不改变任何输入的情况下再次运行该任务:

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fcache-hit.3bac1eb9.png&w=1920&q=75)

1. **哈希值将是相同**的，因为**输入没有改变**（例如：`78awdk123`）

2. Turborepo将在其缓存中找到与计算出的哈希值相同的文件夹（例如：`./node_modules/.cache/turbo/78awdk123`)

3. Turborepo**不会运行任务**，而是**重新输出** - 将保存的日志打印到`stdout`，并将保存的输出文件恢复到文件系统中的相应位置。

从缓存中恢复文件和日志几乎是即时发生的。这可以使你的构建时间从几分钟或几小时减少到几秒钟或几毫秒。虽然具体的结果会因您的代码库的依赖关系图的形状和颗粒度而不同，但大多数团队发现，使用Turborepo的缓存，他们可以将每月的总体构建时间减少40-85%。

## 配置缓存输出 //Configuring Cache Outputs

使用[`管道`](https://turbo.build/repo/docs/reference/configuration#pipeline)，你可以在你的Turborepo上配置缓存约定

要覆盖默认的缓存输出行为，可以向 [`pipeline.<task>.output`](https://turbo.build/repo/docs/reference/configuration#outputs) 数组传递一个 globs 数组。任何满足任务的glob模式的文件都将被视为工件。

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "outputs": ["dist/**", ".next/**"],
      "dependsOn": ["^build"]
    },
    "test": {
      "outputs": [], // leave empty to only cache logs
      "dependsOn": ["build"]
    }
  }
}
```

如果你的任务没有发出任何文件（例如使用Jest的单元测试），将`输出`设置为一个空数组（即`[]`），Turborepo将为你缓存日志。

当你运行 `turbo run build test` 时，Turborepo 会执行你的构建和测试脚本，并将其`输出`缓存在 `./node_modules/.cache/turbo` 中。

缓存ESLint的专业提示：通过在`eslint`前设置`TIMING=1`变量，你可以得到一个可缓存的漂亮的终端输出（即使是非错误的）。在[ESLint文档](https://eslint.org/docs/latest/developer-guide/working-with-rules#per-rule-performance)中了解更多。

## 配置缓存输入 //Configuring Cache Inputs

当一个工作区中的任何文件发生变化时，该工作区被认为已经被更新。然而，对于某些任务，我们只想在相关文件发生变化时重新运行该任务。指定`输入`让我们可以定义哪些文件与特定任务有关。例如，下面的`测试`配置声明，只有当`src/`和`test/`子目录中的`.tsx`或`.ts`文件在上次执行后发生变化时，`测试`任务才需要执行。

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    // ... omitted for brevity

    "test": {
      // A workspace's `test` task depends on that workspace's
      // own `build` task being completed first.
      "dependsOn": ["build"],
      "outputs": [],
      // A workspace's `test` task should only be rerun when
      // either a `.tsx` or `.ts` file has changed.
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"]
    }
  }
}
```

## 关闭缓存 //Turn off caching

有时你真的不想写缓存输出（例如，当你使用 [`next dev`](https://nextjs.org/docs/api-reference/cli#development) 或 `react-scripts start` 进行实时重载时）。要禁用缓存的写入，可以在任何命令后面加上 `--no-cache`:

```shell
# Run `dev` npm script in all workspaces in parallel,
# but don't cache the output
turbo run dev --parallel --no-cache
```

注意，`--no-cache`禁止写缓存，但不禁止读缓存。如果你想禁止缓存的读取，请使用`--force`标志。

你也可以通过将[`pipeline.<task>.cache`](https://turbo.build/repo/docs/reference/configuration#cache)配置为`false`来禁用特定任务上的缓存

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "dev": {
      "cache": false
    }
  }
}
```

## 根据文件变化改变缓存 //Alter Caching Based on File Changes

对于某些任务，如果一个不相关的文件发生了变化，你可能不希望出现缓存错过。例如，更新 `README.md` 可能不需要触发`测试`任务的缓存缺失。你可以用`输入`来限制`turbo`为一个特定任务考虑的文件集。在这种情况下，只考虑与确定`测试`任务的缓存命中有关的`.ts`和`.tsx`文件:

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    // ...other tasks
    "test": {
      "outputs": [], // leave empty to only cache logs
      "dependsOn": ["build"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"]
    }
  }
}
```

`package.json`*总是*被认为是它所在的工作空间中的任务的输入。这是因为任务本身的定义存在于`package.json`的`scripts`键中。如果你改变了这一点，任何缓存的输出都被认为是无效的。

如果你想让 _所有_ 任务都依赖某些文件，你可以在`globalDependencies`数组中声明这种依赖性。

```diff
{
  "$schema": "https://turbo.build/schema.json",
+  "globalDependencies": [".env"],
  "pipeline": {
    // ...other tasks
    "test": {
      "outputs": [], // leave empty to only cache logs
      "dependsOn": ["build"],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts"]
    }
  }
}
```

`turbo.json` *始终*被认为是一个全局依赖。如果你修改了`turbo.json`，所有的缓存都会被废止。

## 根据环境变量改变缓存的方式 //Altering Caching Based on Environment Variables

当你在构建时使用`turbo`和内嵌环境变量的工具（例如[Next.js](https://nextjs.org/docs/basic-features/environment-variables#exposing-environment-variables-to-the-browser)或[Create React App](https://create-react-app.dev/docs/adding-custom-environment-variables/)），告诉`turbo`这一点很重要。否则，你可能会用错误的环境变量发送一个缓存的构建。

你可以根据环境变量的值来控制 `turbo` 的缓存行为。

- 在你的`管道`定义中的`env`键中包含环境变量将影响每个任务或每个工作区任务的缓存指纹。
- 任何在其名称中包含`THASH`的环境变量的值都会影响 _所有_ 任务的缓存指纹。

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      // env vars will impact hashes of all "build" tasks
      "env": ["SOME_ENV_VAR"],
      "outputs": ["dist/**"]
    },

    // override settings for the "build" task for the "web" app
    "web#build": {
      "dependsOn": ["^build"],
      "env": [
        // env vars that will impact the hash of "build" task for only "web" app
        "STRIPE_SECRET_KEY",
        "NEXT_PUBLIC_STRIPE_PUBLIC_KEY",
        "NEXT_PUBLIC_ANALYTICS_ID"
      ],
      "outputs": [".next/**"]
    }
  }
}
```

在 `dependsOn` 配置中用 `$` 前缀来声明环境变量的做法已经过时了。

要改变 _所有_ 任务的缓存，你可以在`globalEnv`数组中声明环境变量:

```diff
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      // env vars will impact hashes of all "build" tasks
      "env": ["SOME_ENV_VAR"],
      "outputs": ["dist/**"]
    },

    // override settings for the "build" task for the "web" app
    "web#build": {
      "dependsOn": ["^build"],
      "env": [
        // env vars that will impact the hash of "build" task for only "web" app
        "STRIPE_SECRET_KEY",
        "NEXT_PUBLIC_STRIPE_PUBLIC_KEY",
        "NEXT_PUBLIC_ANALYTICS_ID"
      ],
      "outputs": [".next/**"],
    },
  },
+  "globalEnv": [
+    "GITHUB_TOKEN" // env var that will impact the hashes of all tasks,
+  ]
}
```

### 自动包含环境变量 //Automatic environment variable inclusion

为了确保在不同环境下的正确缓存，Turborepo在计算使用检测到的框架构建的应用程序的缓存密钥时，自动推断并包括公共环境变量。你可以安全地从`turbo.json`中省略特定框架的公共环境变量:

```diff 
filename="turbo.json"
{
  "pipeline": {
    "build": {
      "env": [
-       "NEXT_PUBLIC_EXAMPLE_ENV_VAR"
      ]
    }
  }
}
```

请注意，只有当Turborepo成功推断出你的应用程序所使用的框架时，这种自动检测和包含才会生效。支持的框架和环境变量，Turborepo将检测并包含在缓存键中。:

- Astro: `PUBLIC_*`
- Blitz: `NEXT_PUBLIC_*`
- Create React App: `REACT_APP_*`
- Gatsby: `GATSBY_*`
- Next.js: `NEXT_PUBLIC_*`
- Nuxt.js: `NUXT_ENV_*`
- RedwoodJS: `REDWOOD_ENV_*`
- Sanity Studio: `SANITY_STUDIO_*`
- Solid: `VITE_*`
- SvelteKit: `VITE_*`
- Vite: `VITE_*`
- Vue: `VUE_APP_*`

上面的列表有一些例外情况。由于各种原因，CI系统（包括Vercel）会设置以这些前缀开头的环境变量，尽管它们不是你构建输出的一部分。这些变量可能会发生不可预知的变化&mdash;甚至在每次构建时都会发生变化&mdash;导致 Turborepo 的缓存失效。为了解决这个问题，Turborepo 使用 `TURBO_CI_VENDOR_ENV_KEY` 变量将环境变量从 Turborepo 的推理中 _排除_。

例如，Vercel 设置了 `NEXT_PUBLIC_VERCEL_GIT_COMMIT_SHA`。这个值在每次构建时都会改变，所以 Vercel _也_ 设置 `TURBO_CI_VENDOR_ENV_KEY="NEXT_PUBLIC_VERCEL_"` 来排除这些变量。

幸运的是，你只需要在其他构建系统上注意这一点；在 Vercel 上使用 Turborepo 时，你不需要担心这些边缘情况。

#### 关于monorepos的说明 //A note on monorepos

环境变量将只包括在使用该框架的工作空间的任务的缓存密钥中。换句话说，为Next.js应用程序推断的环境变量将只包括在被检测为Next.js应用程序的工作空间的缓存密钥中。单元中其他工作空间的任务将不会受到影响。

例如，考虑一个有三个工作空间的monorepo：一个Next.js项目，一个Create React App项目，和一个TypeScript包。每一个都有一个`构建`脚本，而且两个应用都依赖于TypeScript项目。假设这个Turborepo有一个标准的`turbo.json`管道，按顺序构建它们:

```jsonc 
filename="turbo.json"
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"]
    }
  }
}
```

从 1.4 起，当你运行 `turbo run build` 时，Turborepo 在构建 TypeScript 包时不会考虑任何构建时的相关环境变量。然而，在构建 Next.js 应用程序时，Turborepo 会推断出以 `NEXT_PUBLIC_` 开头的环境变量可能会改变 `.next` 文件夹的输出，因此在计算散列值时应该包括在内。同样，在计算 Create React App 的`构建`脚本的哈希值时，所有以 `REACT_APP_PUBLIC_` 开头的构建时间环境变量都将被包括在内。
### `eslint-config-turbo`

为了进一步帮助检测爬到你的构建中的不可见的依赖，并帮助确保你的Turborepo缓存在每个环境中正确共享，使用[`eslint-config-turbo`](https://www.npmjs.com/package/eslint-config-turbo)包。虽然自动包含环境变量应该涵盖大多数框架的大多数情况，这个ESLint配置将为使用其他构建时间内嵌环境变量的团队提供及时的反馈。这也将有助于支持使用我们无法自动检测的内部框架的团队。

现在开始，在你的根目录[`eslintrc`](https://eslint.org/docs/latest/user-guide/configuring/configuration-files#configuration-file-formats)文件中扩展`eslint-config-turbo`:

```jsonc
{
  // Automatically flag env vars missing from turbo.json
  "extends": ["turbo"]
}
```

为了对规则进行更多的控制，你可以直接安装和配置[`eslint-plugin-turbo`](https://www.npmjs.com/package/eslint-plugin-turbo)_插件_，首先将其添加到插件中，然后配置所需的规则。

```jsonc
{
  "plugins": ["turbo"],
  "rules": {
    // Automatically flag env vars missing from turbo.json
    "turbo/no-undeclared-env-vars": "error"
  }
}
```

如果你在代码中使用与框架无关的环境变量，而这些变量没有在你的`turbo.json`中声明，该插件将警告你。

### 无形的环境变量 //Invisible Environment Variables

由于Turborepo在你的任务 _之前_ 运行，所以你的任务有可能在`turbo`已经计算出特定任务的哈希值之后创建或改变环境变量。例如，考虑这个`package.json`:

```json
{
  "scripts": {
    "build": "NEXT_PUBLIC_GA_ID=UA-00000000-0 next build",
    "test": "node -r dotenv/config test.js"
  }
}
```

`turbo`在调用`构建`脚本之前计算了任务哈希值，将无法发现`NEXT_PUBLIC_GA_ID=UA-00000000-0`环境变量，因此无法根据该变量或`dotenv`配置的任何环境变量来划分缓存。

请注意，在调用`turbo`之前，要确保所有的环境变量都已经配置好了!

## 强制覆盖缓存 //Force overwrite cache

相反，如果你想禁止读取缓存，并强迫`turbo`重新执行之前缓存的任务，请添加`--force`标志。:

```shell
# Run `build` npm script in all workspaces,
# ignoring cache hits.
turbo run build --force
```

注意，`--force`会禁止缓存的读取，但不会禁止缓存的写入。如果你想禁止缓存写入，请使用`--no-cache`标志。

## 日志 //Logs

`turbo`不仅会缓存任务的输出，还会将终端输出（即`stdout`和`stderr`的组合）记录到（`<package>/.turbo/run-<command>.log`）。当`turbo`遇到一个缓存的任务时，它将重新播放输出，就像它再次发生一样，但是是即时的，软件包的名字会稍微变暗。

## 哈希 //Hashing

现在，你可能想知道`turbo`是如何决定一个特定任务的缓存命中和未命中的。这是个好问题。

首先，`turbo`构建了一个代码库当前全局状态的哈希值:

- 满足glob模式的任何文件的内容和[`globalDependencies`](https://turbo.build/repo/docs/reference/configuration#globalDependencies)中列出的任何环境变量的值。
- 排序列表中的环境变量键值对，这些键值对在其名称中的 _任何地方_ 都包括`THASH`（例如：`STRIPE_PUBLIC_THASH_SECRET_KEY`，但不是`STRIPE_PUBLIC_KEY`）。

然后，它添加更多与特定工作区的任务相关的因素:

- 工作区文件夹中所有版本控制文件的哈希值，或者与`输入`的globs相匹配的文件，如果存在的话
- 所有内部依赖关系的哈希值
- [`管线`](https://turbo.build/repo/docs/reference/configuration#pipeline)中指定的`输出`选项
- 工作区`package.json`中指定的所有已安装的`dependencies`、`devDependencies`和`optionalDependencies`的解析版本集，这些依赖关系来自根锁定文件
- 工作区任务的名称
- 与适用的 [`pipeline.<task-or-package-task>.dependOn`](https://turbo.build/repo/docs/reference/configuration#dependson) 列表中列出的环境变量名称相对应的环境变量键值对的排序列表。

一旦`turbo`在执行过程中遇到某个工作区的任务，它就会检查缓存（包括本地和远程的）是否有匹配的哈希值。如果是匹配的，它就会跳过执行该任务，将缓存的输出移动或下载到地方，并即时重播之前记录的日志。如果缓存中没有任何东西（无论是本地还是远程）与计算出的哈希值相匹配，`Turbo`将在本地执行任务，然后用哈希值作为索引来缓存指定的`输出`。

一个给定任务的哈希值在执行时被注入环境变量`TURBO_HASH`。这个值在标记输出或标记Dockerfile等方面非常有用。

从`turbo` v0.6.10开始，`turbo`在使用`npm`或`pnpm`时的散列算法与上述略有不同。当使用这两个软件包管理器时，`turbo`会在每个工作区的任务的散列算法中包括锁文件的散列内容。它 _不会_ 像当前的`yarn`实现那样，解析/计算出所有依赖关系的解析集。
