
# 将Turborepo添加到您现有的monorepo中

turbo可以在以下操作系统上与[Yarn](https://classic.yarnpkg.com/lang/en/)、[npm](https://www.npmjs.com/)和[pnpm](https://pnpm.io/)一起工作:

- macOS darwin 64-bit (Intel), ARM 64-bit (Apple Silicon)
- Linux 32-bit, 64-bit, ARM, ARM 64-bit, MIPS 64-bit Little Endian, PowerPC 64-bit Little Endian, IBM Z 64-bit Big Endian
- Windows 32-bit, 64-bit, ARM 64-bit
- FreeBSD 64-bit, ARM 64-bit
- NetBSD AMD64
- Android ARM 64-bit

## 配置工作空间 //Configure workspaces

`turbo`是建立在工作空间的基础上的，工作空间是在一个单一的包中管理多个包的一种方式。Turborepo与所有软件包管理器的工作空间实现兼容。更多关于管理Turborepo工作空间的信息，请看[Workspaces]([/repo/docs/handbook/workspaces](https://turbo.build/repo/docs/handbook/workspaces)) 文档。

你可以以任何方式配置工作空间，但一个常见的文件夹结构的例子是将应用程序放在`/apps`文件夹中，将包放在`/packages`文件夹中。这些文件夹的配置对每个软件包管理器来说都是不同的。

### npm
在monorepo的根目录`package.json`文件中指定你的`workspaces`
```json 
filename="package.json"
{
  "workspaces": ["packages/*", "apps/*"]
}
```

### yarn
在monorepo的根目录`package.json`文件中指定你的`workspaces`
```json 
filename="package.json"
{
  "workspaces": ["packages/*", "apps/*"]
}
```

### pnpm
在`pnpm-workspace.yaml`中指定你的`packages`
```yaml 
filename="pnpm-workspace.yaml"
packages:
  - "packages/*"
  - "apps/*"
```

在配置好你的工作空间后，重新运行你的软件包管理器的`install`命令。

注意：不支持嵌套的工作空间。由于包的名字需要是唯一的，把每个包移到单层包的根包的一个子包应该可以满足你的需要。

## 安装 `turbo`

在你的monorepo的根目录添加`turbo`(https://www.npmjs.com/package/turbo)作为开发依赖项。

### npm
```bash
  npm install turbo -D 
```

### yarn
```bash
  yarn add turbo -DW 
```

### pnpm
```bash
  pnpm add turbo -Dw 
```

`turbo`包是一个shell脚本，它将为你的操作系统和架构安装合适的`turbo-<os>-<arch>`包。

注意：Linux构建的`turbo`与`glibc`链接。对于Alpine Docker环境，你需要确保libc6-compat也被安装，通过`RUN apk add --no-cache libc6-compat`

## 创建 `turbo.json`

在你的 monorepo 根目录下，创建一个名为 `turbo.json` 的空文件。这将保存Turborepo的配置。

```json 
filename="./turbo.json"
{
  "$schema": "https://turbo.build/schema.json"
}
```

## 创建 `pipeline`

要定义你的monorepo的任务依赖图，使用monorepo根部的`turbo.json`配置文件中的[管道](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)键。`turbo`解释这个配置，以最佳方式安排、执行和缓存你的工作空间中定义的每个`package.json`脚本的输出。

[管道](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)对象中的每个键都是`package.json`脚本的名称，可以通过[`turbo run`](https://turbo.build/repo/docs/reference/command-line-reference#turbo-run-task)执行。你可以用里面的dependOn键指定它的依赖关系，以及其他一些与[缓存](https://turbo.build/repo/docs/core-concepts/caching)有关的选项。关于配置管道的更多信息，请参阅[管道](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)文档。

如果工作空间的`package.json`的`脚本`列表中没有定义指定的脚本，那么`turbo`就会忽略掉。

```jsonc 
filename="./turbo.json"
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      // A package's `build` script depends on that package's
      // dependencies and devDependencies
      // `build` tasks  being completed first
      // (the `^` symbol signifies `upstream`).
      "dependsOn": ["^build"],
      // note: output globs are relative to each package's `package.json`
      // (and not the monorepo root)
      "outputs": [".next/**"]
    },
    "test": {
      // A package's `test` script depends on that package's
      // own `build` script being completed first.
      "dependsOn": ["build"],
      "outputs": [],
      // A package's `test` script should only be rerun when
      // either a `.tsx` or `.ts` file has changed in `src` or `test` folders.
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts", "test/**/*.tsx"]
    },
    "lint": {
      // A package's `lint` script has no dependencies and
      // can be run whenever. It also has no filesystem outputs.
      "outputs": []
    },
    "deploy": {
      // A package's `deploy` script depends on the `build`,
      // `test`, and `lint` scripts of the same package
      // being completed. It also has no filesystem outputs.
      "dependsOn": ["build", "test", "lint"],
      "outputs": []
    }
  }
}
```

一个给定包的粗略执行顺序是基于 `dependsOn` 键:

1. `build` 一旦它的上游依赖项运行了它们的`build`命令。
2. `test`  一旦它 _自己的_`build`命令完成，并且在一个包内没有文件系统输出（只有日志）。
3. `lint`  以任意的顺序运行，因为它没有上游的依赖性
4. `deploy` 一旦它 _自己的_`build`、`test`和`lint`命令完成，就进行部署。

执行后，整个管道可以运行:

```sh
npx turbo run build test lint deploy
```

然后，`Turbo`将安排每个任务的执行，以优化机器的资源使用。

## 编辑 `.gitignore`

在你的`.gitignore`文件中添加`.turbo`。CLI使用这些文件夹来记录日志和某些任务输出。

```diff
+ .turbo
```

确保你的任务工件，即你想要缓存的文件和文件夹，也包含在你的`.gitignore`中。

```diff
+ build/**
+ dist/**
+ .next/**
```

重新运行npm客户端的`install`命令来检查你的配置。

## 创建 `package.json` 脚本

在你的monorepo的根目录`package.json`文件中添加或更新`脚本`，让它们委托给`turbo`。

```jsonc 
filename="./package.json"
{
  "scripts": {
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "dev": "turbo run dev"
  }
}
```

## 构建你的 monorepo

### npm
```sh
  npx run build 
```

### yarn
```sh
  yarn build 
```

### pnpm
```sh
  pnpm run build 
```

根据你的monorepo设置，一些人工制品可能已经在正常缓存了。在接下来的章节中，我们将展示`Turbo`是如何工作的，`范围`是如何工作的，以及之后如何让缓存工作。

## 配置远程缓存

Turborepo的速度的一个主要关键🔑是它既懒惰又高效--它尽可能做最少的工作，并且尽量不重做以前已经做过的工作。

目前，Turborepo将你的任务缓存在你的本地文件系统中（即 "单人模式"，如果你愿意）。然而，如果有一种方法可以利用你的队友或你的CI所做的计算工作（即 "多人合作模式"）呢？如果有一种方法可以在机器间传送和共享单一的缓存呢？几乎就像为你的Turborepo缓存提供一个 "Dropbox"。

> Remote Caching has entered the chat.

Turborepo可以使用一种被称为 "远程缓存 "的技术，在不同的机器上共享缓存工件，以获得额外的速度提升。

远程缓存是Turborepo的一个强大功能，但强大的功能也伴随着巨大的责任。首先确保你的缓存是正确的，并仔细检查环境变量的处理。也请记住Turborepo将日志作为人工制品处理，所以要注意你在控制台打印的内容。

### 使用远程缓存进行本地开发 //Using Remote Caching for Local development

Turborepo使用[Vercel](https://vercel.com/?utm_source=turbo.build&utm_medium=referral&utm_campaign=docs-link)作为其默认的远程缓存提供者。如果你想把你的本地turborepo和你的远程缓存联系起来，你可以用你的Vercel账户来验证Turborepo CLI:

```sh
npx turbo login
```

然后，将你的turborepo链接到你的远程缓冲区:

```
npx turbo link
```

一旦启用，对你目前正在缓存的软件包或应用程序做一些修改，然后用`turbo run`对其运行任务。现在你的缓存工件将被保存在本地和你的远程缓存中。为了验证这是否有效，请删除你的本地Turborepo缓存:

```sh
rm -rf ./node_modules/.cache/turbo
```

再次运行相同的构建。如果工作正常，`Turbo`不应该在本地执行任务，而是从你的远程缓存中下载日志和工件，并将它们回放给你。

**注意：当连接到一个启用了sso的Vercel团队时，你必须提供你的团队slug作为`npx turbo login`的参数。**

```
npx turbo login --sso-team=<team-slug>
```

## 接下来的步骤

你现在已经开始使用Turborepo了，但是还有一些事情要做:

- [理解Turborepo缓存的工作原理](https://turbo.build/repo/docs/core-concepts/caching)
- [正确地处理环境变量](https://turbo.build/repo/docs/core-concepts/caching#altering-caching-based-on-environment-variables)
- [学会用管道协调任务的运行](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)
- [有效地过滤软件包任务](https://turbo.build/repo/docs/core-concepts/monorepos/filtering)
- [将Turborepo与您的CI供应商进行配置](https://turbo.build/repo/docs/ci)
