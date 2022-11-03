# Command-Line 参考资料

After [installing the `turbo`](../getting-started/add-to-project) package (or cloning a starter), you can start using Turborepo's command line interface (CLI) `turbo` to do all kinds of awesomeness in your monorepo.

[安装完`turbo`](https://turbo.build/repo/docs/getting-started/add-to-project)包（或克隆一个启动器）后，你可以开始使用Turborepo的命令行界面（CLI）`turbo`来在你的monorepo中做各种精彩的事情。

## 选项语法 //Option Syntax

选项可以用不同的方式传递给`turbo`。需要一个值的选项可以用等号来传递:

```sh
--opt=<value>
--opt="<value with a space>"
```

它们之间也可以用一个空格来传递:

```sh
--opt value
--opt "value with a space"
```

布尔选项可按以下方式启用:

```sh
# To pass true
--opt

# To pass false
--opt=false
```

## 全局参数(实参)//Global Arguments

以下标志适用于所有命令.

#### `--color`

即使输出流不被认为是TTY终端，也会强制使用颜色。这可以用来为CI运行程序启用`turbo`的颜色输出，比如Github Actions，它支持在其日志输出中呈现颜色。

```sh
turbo run build --color
```

另外，你也可以使用 `FORCE_COLOR` 环境变量（从 [supports-color nodejs 包](https://www.npmjs.com/package/supports-color)中借用）来启用颜色。请注意，如果实际任务使用 `supports-color` 来决定是否使用彩色输出，这也可能使实际任务本身产生额外的彩色输出。

```sh
declare -x FORCE_COLOR=1
turbo run build
```

#### `--no-color`

当在interactive/TTY会话中运行`Turbo`时，禁止在输出中使用颜色。

```sh
turbo run build --no-color
```

另外，你也可以使用`FORCE_COLOR`环境变量（从[support-color nodejs包](https://www.npmjs.com/package/supports-color)中借用）来抑制颜色。

```sh
declare -x FORCE_COLOR=0
turbo run build
```

## `turbo run <task>`

在指定范围内的所有工作空间运行npm脚本。任务必须在你的`管道`配置中指定。

`turbo run <task1> <task2> [options] [-- <args passed to task1 and task2>]`

`turbo`可以运行多个任务，任何在`--`后面的参数将被传递到要执行的任务中。请注意，这些额外的参数将 _不_ 会被传递给任何由于[管道](https://turbo.build/repo/docs/reference/configuration#pipeline)配置的依赖性而被运行的额外任务。

### 选项 //Options

#### `--cache-dir`

`type: string`

默认为`./node_modules/.cache/turbo`。指定本地文件系统的缓存目录。如果你改变了默认值，请确保将此文件夹添加到你的`.gitignore`中。

```sh
turbo run build --cache-dir="./my-cache"
```

#### `--concurrency`

`type: number | string`

默认为`10`。设置/限制任务执行的最大并发量。这必须是一个大于或等于`1`的整数或一个百分比值，如`50%`。使用`1`来强制串行（即一次一个任务）执行。使用`100%`来使用所有可用的逻辑处理器。如果同时通过了[`--parallel`](https://turbo.build/repo/docs/reference/command-line-reference#--parallel)标志，这个选项将被忽略。

```sh
turbo run build --concurrency=50%
turbo run test --concurrency=1
```

#### `--continue`

默认为`false`。这个标志告诉`turbo`在出现错误（即任务的非零退出代码）时是否继续执行。默认情况下，除非明确设置为false，否则指定`--parallel`标志会自动将`--continue`设置为`true`。当`--continue`为`true`时，`turbo`将以执行过程中遇到的最高退出代码值退出。

```sh
turbo run build --continue
```

#### `--cwd`

设置命令的工作目录。

```sh
turbo run build --cwd=./somewhere/else
```

#### `--deps`

在`1.2.x`版本中，`--deps`已被弃用，请使用[`--filter`](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#include-dependents-of-matched-packages) 代替

默认为`true`。在执行中包括依赖的工作区消费者。

```sh
turbo run build --deps
turbo run build --no-deps
```

**例子**

假设你有 A、B、C 和 D 工作区，其中 A 依赖于 B，C 依赖于 D。你第一次运行 `turbo run build`，所有东西都被构建和缓存了。然后，你修改了 B 中的一行代码。如果打开`--deps`标志，运行 `turbo run build` 会在 B 中执行构建，然后是 A，但不会在 C 和 D 中执行，因为它们不受修改的影响。如果你运行`turbo run build --no-deps`，turbo将只在B中运行`build`。

#### `--dry / --dry-run`

显示关于受影响的工作空间和将被运行的任务的细节,而不是执行任务。指定`--dry=json`来获得JSON格式的输出。

Task 细节包括:

- `task`: 要执行的任务的名称
- `package`: 运行任务的工作区
- `hash`: 任务的哈希值，用于缓存
- `directory`: 将要运行任务的目录
- `command`: 用来运行任务的实际命令
- `outputs`: 将被缓存的任务输出的位置
- `logFile`: 任务运行的日志文件的位置
- `dependencies`: 必须在此任务之前运行的任务
- `dependents`: 在此任务之后必须运行的任务

#### `--filter`

`type: string[]`

指定工作空间、目录和git提交的组合，作为执行的入口。

多个过滤器可以结合起来，选择不同的目标集。此外，过滤器还可以排除目标。匹配任何过滤器且未明确排除的目标将被列入最终的入口选择。

关于`--filter`标志和过滤的更多详细信息，请参考[我们文档中的专门页面](https://turbo.build/repo/docs/core-concepts/monorepos/filtering)

```sh
turbo run build --filter=my-pkg
turbo run test --filter=...^@scope/my-lib
turbo run build --filter=./apps/* --filter=!./apps/admin
```

#### `--graph`

该命令将生成一个svg、png、jpg、pdf、json、html或[其他支持的输出格式](https://graphviz.org/doc/info/output.html)在当前任务视图。输出文件格式默认为jpg，但可以通过指定文件名的扩展名来控制。

如果Graphviz没有安装，或者没有提供文件名，这个命令会将点状图打印到`stdout`。

```sh
turbo run build --graph
turbo run build test lint --graph=my-graph.svg
turbo run build test lint --graph=my-json-graph.json
turbo run build test lint --graph=my-graph.pdf
turbo run build test lint --graph=my-graph.png
turbo run build test lint --graph=my-graph.html
```

**已知BUG**: 所有可能的管道任务节点将被添加到图中，即使该管道任务实际上不存在于给定的工作空间中。这对执行没有影响，它意味着1) 终端输出可能会夸大一个任务正在运行的工作空间的数量，并且2)你的dot viz图可能包含额外的节点，代表不存在的任务。

#### `--force`

忽略现有的缓存工件，强行重新执行所有任务（覆盖重叠的工件）。

```sh
turbo run build --force
```

同样的行为也可以通过`TURBO_FORCE=true`环境变量设置。

#### `--global-deps`

指定全局文件系统的依赖关系的glob来进行哈希。对于影响多个packages/apps的根目录中的.env和文件很有用。可以指定多次。

```sh
turbo run build --global-deps=".env.*" --global-deps=".eslintrc" --global-deps="jest.config.js"
```

你也可以在你的`turbo`配置中指定这些作为`globalDependencies`键。

#### `--ignore`

`type: string[]`

忽略**文件或目录**对范围的影响。在底层使用glob模式。

```
turbo run build --ignore="apps/**/*"
turbo run build --ignore="packages/**/*"
turbo run build --ignore="packages/**/*" --ignore="\!/packages/not-this-one/**/*"
```

##### 多种模式如何运作 //How multiple patterns work

正面模式（如`foo`或`*`）增加结果，而负面模式（如`!foo`）从结果中减去。

因此，一个单独的否定词（例如`['!foo']`）永远不会匹配任何东西--使用`['*'，'!foo']`代替。

##### 全局模式 //Globbing patterns

只是一个简单的概述。

- `*` 匹配任何数量的字符, 但不匹配 `/`
- `?` 匹配单个字符, 但不匹配 `/`
- `**` 匹配任何数量的字符, 包括 `/`, 只要它是路径部分的唯一字符
- `{}` 允许使用以逗号分隔的 "or"表达式列表
- `!` 在一个模式的开头出现，将否定该匹配

#### `--include-dependencies`

在`1.2.x`版本中，`--include-dependencies`已被弃用，请使用[`--filter`](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#include-dependents-of-matched-packages) 代替

默认为`false`。当为`true`时，`turbo`将添加当前执行中的工作空间所 _依赖_ 的任何工作空间（即那些在`dependencies`或`devDependencies`中声明的工作空间）。

这在CI中使用`--filter`时很有用，因为它保证执行所需的每一个依赖都被实际执行。

#### `--no-cache`

默认为`false`。不缓存任务的结果。这对于像`next dev`或`react-scripts start`这样的观察命令很有用。

```shell
turbo run build --no-cache
turbo run dev --parallel --no-cache
```

#### `--no-daemon`

默认为`false`。在某些情况下，`turbo`可以运行一个独立的进程来预先计算用于确定需要做什么工作的数值。这个独立的进程（daemon）是一种优化，并不是`turbo`正常运行所必需的。传递`--no-daemon`指示`turbo`避免使用或创建这个独立的进程。

#### `--output-logs`

`type: string`

设置输出日志的类型。对于`turbo.json`中的任务，默认为 "outputMode"。

|  option   | description  |
|  ----  | ----  |
| full  | 这是默认的。显示所有输出 |
| hash-only  | 只显示任务的哈希值 |
| new-only  | 只显示缓存缺失的输出 |
| none  | 隐藏所有的任务输出 |

**例子**

```shell
turbo run build --output-logs=full
turbo run build --output-logs=new-only
```

#### `--only`

默认为`false`。限制执行，只包括指定的任务。这与 `lerna` 和 `pnpm` 默认运行任务的方式非常相似。

在`turbo.json`中给出这个管道:

```json
{
  "$schema": "https://turborepo.org/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["^build"]
    }
  }
}
```

```shell
turbo run test --only
```

将 _只_ 执行每个工作区的`test`任务。它不会`build`。

#### `--parallel`

默认为`false`。跨工作区并行运行命令，忽略依赖关系图。这在开发实时重载时很有用。

```sh
turbo run lint --parallel --no-cache
turbo run dev --parallel --no-cache
```

#### `--remote-only`

默认为`false`。忽略所有任务的本地文件系统缓存。只允许使用远程缓存读取和缓存工件。

```shell
turbo run build --remote-only
```

同样的行为也可以通过`TURBO_REMOTE_ONLY=true`环境变量设置。

#### `--scope`

在`1.2.x`版本中，`--scope`已被弃用，请使用[`--filter`](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#include-dependents-of-matched-packages) 代替

`type: string[]`

Specify/filter 工作空间，作为执行的入口点。根据`package.json`的`name`字段（而不是文件系统）进行Globs。

```sh
turbo run lint --scope="@example/**"
turbo run dev --parallel --scope="@example/a" --scope="@example/b" --no-cache --no-deps
```

#### `--serial`

在`0.5.3`版本中，`serial`已被弃用，请使用[`--concurrency=1`](https://turbo.build/repo/docs/reference/command-line-reference#--concurrency) 代替

串行地执行所有任务（即一次一个）。

```sh
turbo run build --serial
```

#### `--since`

在`1.2.x`版本中，`--since`已被弃用，请使用[`--filter`](https://turbo.build/repo/docs/core-concepts/monorepos/filtering#include-dependents-of-matched-packages) 代替

根据哪些工作空间在合并基础后发生了变化来过滤执行。

```
turbo run build --since=origin/main
```

**重点**: 这使用`git diff ${target_branch}...`机制来识别哪些工作区有变化。这里有一个假设，即一个工作区的所有输入文件都存在于各自的工作区文件夹中。

#### `--token`

一个用于远程缓存的承载令牌。对于在非交互式shells（如CI/CD）中运行，与`--team`标志结合使用非常有用。

```sh
turbo run build --team=my-team --token=xxxxxxxxxxxxxxxxx
```

你也可以通过设置一个名为`TURBO_TOKEN`的环境变量来设置当前令牌的值。如果两者都存在，该标志将优先于环境变量。


如果你在Vercel上使用远程缓存并在Vercel上构建你的项目，这个环境变量和标志就没有必要了，因为它们会自动为你设置。假设你在Vercel上使用远程缓存，但在另一个CI提供商（如CircleCI或GitHub Actions）中构建。你可以使用Vercel个人访问令牌作为你的`--token`或`TURBO_TOKEN`。如果你使用自定义的远程缓存，这个值将被用来发送HTTP承载器令牌，并向你的自定义远程缓存请求。

#### `--team`

远程缓存团队的名称。在非交互式shell中运行时，与`--token`和`--team`标志结合使用非常有用。

```sh
turbo run build --team=my-team
turbo run build --team=my-team --token=xxxxxxxxxxxxxxxxx
```

你也可以通过设置一个名为`TURBO_TEAM`的环境变量来设置当前队伍的数值。如果两者都存在，该标志将优先于环境变量。

#### `--preflight`

只适用于配置了远程工件缓存的情况。启用在每个缓存工件和分析请求之前发送一个预检请求。后续的上传和下载将遵循重定向。

```sh
turbo run build --preflight
```

同样的行为也可以通过`TURBO_PREFLIGHT=true`环境变量设置。

#### `--trace`

`type: string`

要查看CPU跟踪，将跟踪结果输出到给定的文件中，使用`go tool trace [file]`。

**重要**: 跟踪查看器在Windows Subsystem for Linux下不工作

```sh
turbo run build --trace="<trace-file-name>"
```

#### `--heap`

`type: string`

要查看堆跟踪，把跟踪输出到给定的文件中，使用`go tool pprof [file]`，然后输入`top`。你也可以把它放到[speedscope](https://speedscope.app)中，使用`left heavy` 或 `sandwich`视图模式。

```sh
turbo run build --heap="<heap-file-name>"
```

#### `--cpuprofile`

`type: string`

要查看CPU配置文件，请将配置文件输出到给定的文件中，并将该文件放入[speedscope](https://speedscope.app)中。

**重要**: CPU分析器在Windows Subsystem for Linux下不工作。剖析器必须为本地Windows建立，并使用命令提示符来运行。

```sh
turbo run build --cpuprofile="<cpu-profile-file-name>"
```

#### `-v`, `-vv`, `-vvv`

要指定日志级别，使用`-v`代表`Info`，`-vv`代表`Debug`，`-vvv`代表 `Trace`标志。

```sh
turbo run build -v
turbo run build -vv
turbo run build -vvv
```

## `turbo prune --scope=<target>`

为目标工作区生成一个带有修剪过的锁定文件的 sparse/partial 单核程序。
这个命令还没有在`npm`上实现。

这条命令将`输出`一个文件夹，其中包含以下内容
- 构建目标所需的所有内部工作空间的完整源代码
- 一个新的修剪过的锁文件，只包含原始根锁文件的修剪过的子集，其中的依赖关系是被修剪过的工作区实际使用的。
- 根目录`package.json`的副本

```
.                                 # 1.Folder full source code for all workspaces needed to build the target
├── package.json                  # 3.The root `package.json`
├── packages
│   ├── ui
│   │   ├── package.json
│   │   ├── src
│   │   │   └── index.tsx
│   │   └── tsconfig.json
│   ├── shared
│   │   ├── package.json
│   │   ├── src
│   │   │   ├── __tests__
│   │   │   │   ├── sum.test.ts
│   │   │   │   └── tsconfig.json
│   │   │   ├── index.ts
│   │   │   └── sum.ts
│   │   └── tsconfig.json
│   └── frontend
│       ├── next-env.d.ts
│       ├── next.config.js
│       ├── package.json
│       ├── src
│       │   └── pages
│       │       └── index.tsx
│       └── tsconfig.json
└── yarn.lock                            # 2.The pruned lockfile for all targets in the subworkspace
```

### 选项

#### `--docker`

`type: boolean`

默认为`false`。传递这个标志将改变输出的文件夹与修剪后的工作空间，使其更容易与[Docker最佳实践/层缓存](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)一起使用。

使用`--docker`标志。`prune`命令将`输出`文件夹，里面有以下内容的调用

- 包含修剪后的工作区`package.json`们的文件夹`json`
- 一个`full`修剪后的工作区完整源代码的文件夹，但只包括构建目标所需的内部包。
- 一个新的修剪过的锁文件，只包含原始根锁文件的修剪过的子集，其中的依赖关系是被修剪过的工作区中的软件包实际使用的。

```
.
├── full                                # Folder full source code for all package needed to build the target
│   ├── package.json
│   └── packages
│       ├── ui
│       │   ├── package.json
│       │   ├── src
│       │   │   └── index.tsx
│       │   └── tsconfig.json
│       ├── shared
│       │   ├── package.json
│       │   ├── src
│       │   │   ├── __tests__
│       │   │   │   ├── sum.test.ts
│       │   │   │   └── tsconfig.json
│       │   │   ├── index.ts
│       │   │   └── sum.ts
│       │   └── tsconfig.json
│       └── frontend
│           ├── next-env.d.ts
│           ├── next.config.js
│           ├── package.json
│           ├── src
│           │   └── pages
│           │       └── index.tsx
│           └── tsconfig.json
├── json                                # Folder containing just package.jsons for all targets in the subworkspace
│   ├── package.json
│   └── packages
│       ├── ui
│       │   └── package.json
│       ├── shared
│       │   └── package.json
│       └── frontend
│           └── package.json
└── yarn.lock                           # The pruned lockfile for all targets in the subworkspace
```

## `turbo login`

将机器连接到你的远程缓存提供者。默认的提供者是[Vercel](https://vercel.com)

### 选项

#### `--url`

`type: string`

默认为`https://vercel.com`。

#### `--api`

`type: string`

默认为`https://vercel.com/api`。

#### `--sso-team`

`type: string`

通过提供你的团队编号，连接到一个启用了sso的Vercel团队。

```
turbo login --sso-team=<team-slug>
```

## `turbo logout`

将你从你的Vercel账户中注销。

## `turbo link`

将当前目录链接到远程缓存范围。被选中的所有者（用户或和组织）将能够通过[远程缓存](https://turbo.build/repo/docs/core-concepts/remote-caching)共享[缓存工件](https://turbo.build/repo/docs/core-concepts/caching)。你应该从你的monorepo的根部运行这个命令。

### Options

#### `--api`

`type: string`

默认为`https://api.vercel.com`。

## `turbo unlink`

从远程缓存中解除对当前目录的链接。

## `turbo bin`

Get the path to the `turbo` binary.
获得通往`turbo`二进制的路径。