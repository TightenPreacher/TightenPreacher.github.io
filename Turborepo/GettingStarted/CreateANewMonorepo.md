
# 创建一个新的monorepo //Creating a new monorepo

## 快速入门

要创建一个新的monorepo，使用我们的[`create-turbo`](https://www.npmjs.com/package/create-turbo)npm包

```sh
npx create-turbo@latest
```

你也可以克隆一个Turborepo启动库，为你的monorepo开个头。要看Turborepo的例子和启动程序，请看[GitHub上的Turborepo例子目录](https://github.com/vercel/turbo/tree/main/examples)。

## 完整教程

本教程将指导你使用Turborepo的[`基本`例子](https://github.com/vercel/turbo/tree/main/examples/basic)。到最后，你会对使用Turbo充满信心，并知道所有的基本功能。

在本教程中，一些代码行从代码样本中被省略了。例如，当显示`package.json`时，我们不会显示 _所有_ 的键-只有那些重要的键。

### 1. 运行 `create-turbo`

第一, 运行:

```sh
npx create-turbo@latest
```

这就安装了[create-turbo](https://www.npmjs.com/package/create-turbo) CLI，并运行它。你会被问到几个问题

#### 你想在哪里创建你的Turborepo //Where would you like to create your turborepo?

选择你喜欢的任何地方。默认是`./my-turborepo`

#### 你想使用哪个软件包管理器 //Which package manager do you want to use?

Turborepo并不处理安装软件包，所以你需要选择其中之一

- [npm](https://www.npmjs.com//)
- [pnpm](https://pnpm.io/)
- [yarn](https://yarnpkg.com/)

如果你不确定，我们建议选择`pnpm`。如果你没有安装它，取消`create-turbo`（通过`ctrl-C`），看看[安装说明](https://pnpm.io/installation)

#### 安装 //Installation

一旦你选择了一个软件包管理器，`create-turbo`将在你选择的文件夹中创建一堆新的文件。它还会默认安装`基本`例子中的所有依赖项

### 2. 探索你的新 repo //Exploring your new repo

你可能已经注意到终端中的一些东西。 `create-turbo`给你描述了它正在添加的所有东西。

```
>>> Creating a new turborepo with the following:

 - apps/web: Next.js with TypeScript
 - apps/docs: Next.js with TypeScript
 - packages/ui: Shared React component library
 - packages/eslint-config-custom: Shared configuration (ESLint)
 - packages/tsconfig: Shared TypeScript `tsconfig.json`
```

每一个都是一个 _工作空间_ -一个包含`package.json`的文件夹。每个工作空间可以声明自己的依赖关系，运行自己的脚本，并导出代码供其他工作空间使用。

Open the root folder - `./my-turborepo` - in your favourite code editor.
在你喜欢的代码编辑器中打开根文件夹-`./my-turborepo`。

#### 了解 `packages/ui`

首先，打开`./packages/ui/package.json`。你会注意到包的名字是 `"name": "ui"` - 就在文件的顶部。

接下来，打开`./apps/web/package.json`。你会注意到，这个包的名字是 `"name": "web"`。但同时--看一下它的依赖关系。

你会看到，`"web"`依赖于一个叫做`"ui"`的包。如果你使用的是`pnpm`，你会看到它是这样声明的:

```json 
filename="apps/web/package.json"
{
  "dependencies": {
    "ui": "workspace:*"
  }
}
```

这意味着我们的**网络应用依赖于我们的本地`ui`包**。

如果你看看`apps/docs/package.json`里面，你会看到同样的情况。`web`和`docs`都依赖于`ui`-一个共享组件库。

这种跨应用程序共享代码的模式在monorepos中极为常见-这意味着多个应用程序可以共享一个设计系统。

#### 了解导入和导出

看一下`./apps/docs/pages/index.tsx`里面。`docs`和`web`都是[Next.js](https://nextjs.org/) 应用程序，它们都以类似的方式使用`ui`库

```tsx 
filename="apps/docs/pages/index.tsx"
import { Button } from "ui";
//       ^^^^^^         ^^

export default function Docs() {
  return (
    <div>
      <h1>Docs</h1>
      <Button />
    </div>
  );
}
```

他们直接从一个叫`ui`的依赖关系中导入`Butto`n!那是怎么做到的？`Button`是从哪里来的？

打开 `packages/ui/package.json`。你会注意到这两个属性:

```json 
filename="packages/ui/package.json"
{
  "main": "./index.tsx",
  "types": "./index.tsx"
}
```

当工作空间从`ui`导入时，`main`告诉他们在哪里访问他们要导入的代码。 `types`告诉他们TypeScript类型的位置。

所以，让我们看看`packages/ui/index.tsx`里面:

```tsx 
filename="packages/ui/index.tsx"
import * as React from "react";
export * from "./Button";
```

这个文件中的所有内容都将能够被依赖`ui`的工作空间使用。

`index.tsx`从一个叫做`./Button`的文件中导出所有东西，所以我们去那里:

```tsx 
filename="packages/ui/Button.tsx"
import * as React from "react";

export const Button = () => {
  return <button>Boop</button>;
};
```

我们已经找到了我们的按钮!我们在这个文件中的任何改动都会在`web` 和 `docs`中共享。相当酷!

试着从这个文件中导出一个不同的函数来进行实验。也许可以用`add(a, b)`将两个数字相加。

然后，这可以由`web` 和 `docs`导入。

#### 了解 `tsconfig`
我们还有两个工作空间要看，`tsconfig`和`eslint-config-custom`。这两个工作区都允许在整个monorepo中进行共享配置。让我们来看看`tsconfig`的情况。

```json 
filename="packages/tsconfig/package.json"
{
  "name": "tsconfig",
  "files": ["base.json", "nextjs.json", "react-library.json"]
}
```

在这里，我们指定了三个要导出的文件，在`文件`里面。然后，依赖`tsconfig`的软件包可以直接导入它们。

例如，`packages/ui` 依赖于 `tsconfig`

```json 
filename="packages/ui/package.json"
{
  "devDependencies": {
    "tsconfig": "workspace:*"
  }
}
```

在它的`tsconfig.json`文件中，它使用 `extends`来导入它:

```json 
filename="packages/ui/tsconfig.json"
{
  "extends": "tsconfig/react-library.json"
}
```

这种模式允许单机版在其所有工作空间中共享一个`tsconfig.json`，减少代码的重复

#### 了解 `eslint-config-custom`

我们最后的工作空间是`eslint-config-custom`。

你会注意到它的命名与其他工作区略有不同。它不像`ui`或`tsconfig`那样简明。让我们来看看在monorepo根目录的`.eslintrc.js`中的内容，以找出原因

```ts filename=".eslintrc.js"
module.exports = {
  // This tells ESLint to load the config from the workspace `eslint-config-custom`
  extends: ["custom"],
};
```

[ESLint](https://eslint.org/)通过寻找名称为`eslint-config-*`的工作空间来解析配置文件。这让我们可以写`extends:['custom']` 并让ESLint找到我们的本地工作空间。

但是，为什么这个文件在monorepo的根目录下?

ESLint找到它的配置文件的方式是通过寻找最接近的`.eslintrc.js`。如果它在当前目录下找不到，它就会在上面的目录下寻找，直到找到为止

因此，这意味着如果我们在`packages/ui`中工作的代码（它没有一个`.eslintrc.js`），它就会引用 _根目录_ 来代替

有`.eslintrc.js`的应用程序可以以同样的方式引用`custom`。例如，在`docs`中

```ts filename="apps/docs/.eslintrc.js"
module.exports = {
  root: true,
  extends: ["custom"],
};
```

就像`tsconfig`一样，`eslint-config-custom`让我们在整个monorepo中共享ESLint配置，无论你在哪个项目上工作，都能保持一致。

#### 摘要

了解这些工作空间之间的依赖关系很重要。让我们把它们绘制出来:

{/* Could be worth a diagram here? */}

- `web` - 依赖于`ui`、`tsconfig`和`eslint-config-custom`
- `docs` - 依赖于`ui`、`tsconfig`和`eslint-config-custom`
- `ui` - 依赖于`tsconfig`和`eslint-config-custom`
- `tsconfig` - 无依赖性
- `eslint-config-custom` - 无依赖性

注意，**Turborepo CLI 不负责管理这些依赖关系**。上面所有的事情都由你选择的软件包管理器（`npm`、`pnpm`或`yarn`）来处理。

### 3. 了解 `turbo.json`

我们现在了解了我们的版本库和它的依赖关系。Turborepo如何帮助我们?

Turborepo的帮助是使运行任务更简单、更高效。

让我们看看根目录`package.json`的内容:

```json 
filename="package.json"
{
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint"
  }
}
```

我们在这里有三个使用`turbo run`的`scripts`指定的任务。你会注意到，每个任务都在`turbo.jso`n中被指定。

```json 
filename="turbo.json"
{
  "pipeline": {
    "build": {
      //   ^^^^^
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
    },
    "lint": {
      //   ^^^^
      "outputs": []
    },
    "dev": {
      //   ^^^
      "cache": false
    }
  }
}
```

我们在这里看到的是，我们在 `turbo` _注册_ 了三个任务-`lint`、`dev` 和 `build`。每个在 `turbo.json` 中注册的任务都可以用 `turbo run <task>` 运行。

让我们在根目录`package.json`中添加一个脚本，来看看它的实际效果吧:

```diff 
filename="package.json"
{
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev --parallel",
    "lint": "turbo run lint",
+   "hello": "turbo run hello"
  }
}
```

现在，让我们运行 `hello`.

### npm
```bash
  npm run hello
```

### yarn
```bash
  yarn hello
```

### pnpm
```bash
  pnpm run hello
```

你会在控制台看到这个错误:

```
task `hello` not found in turbo `pipeline` in "turbo.json".
Are you sure you added it?
```

这一点值得记住-**为了让`turbo`运行一个任务，它必须在`turbo.json`中出现**

让我们研究一下我们已经有的脚本.

### 4. Linting with Turborepo

尝试运行我们的 `lint` 脚本:

### npm
```bash
  npm run lint
```

### yarn
```bash
  yarn lint
```

### pnpm
```bash
  pnpm run lint
```

你会注意到在终端发生的几件事。

1. 几个脚本将同时运行，每个脚本的前缀是`docs:lint`, `ui:lint` 或 `web:lint`。
2. 他们会各自成功，你会在终端看到`3个成功`。
3. 你还会看到`0个缓存，共3个`。我们稍后将介绍这意味着什么。

每个运行的脚本都来自每个工作区的`package.json`。每个工作区可以选择指定自己的`lint`脚本:

```json 
filename="apps/web/package.json"
{
  "scripts": {
    "lint": "next lint"
  }
}
```

```json 
filename="apps/docs/package.json"
{
  "scripts": {
    "lint": "next lint"
  }
}
```

```json 
filename="packages/ui/package.json"
{
  "scripts": {
    "lint": "eslint *.ts*"
  }
}
```

当我们运行`turbo run lint`时，Turborepo会查看每个工作区的每个`lint`脚本并运行它。更多细节，请看我们的[管道文档](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks#defining-a-pipeline)。

#### 使用缓存 //Using the cache

让我们再运行一次我们的`lint`脚本。你会发现终端中出现了一些新的东西

1. `cache hit`, `replaying output`出现在`docs:lint`, `web:lint` 和 `ui:lint`。
2. 你会看到`3个缓存`，`总共3个。
3. 总的运行时间应该在`100ms`以下，并且出现`>>>FULL TURBOThe`。

有趣的事情刚刚发生。Turborepo意识到**我们的代码在上次运行lint脚本后没有改变**。

它保存了上一次运行的日志，所以它只是重新播放了这些日志。

让我们试着改变一些代码，看看会发生什么。对`app/docs`中的一个文件进行修改

```diff filename="apps/docs/pages/index.tsx"
import { Button } from "ui";

export default function Docs() {
  return (
    <div>
-     <h1>Docs</h1>
+     <h1>My great docs</h1>
      <Button />
    </div>
  );
}
```

现在，再次运行`lint`脚本。你会注意到:

1. `docs:lint` 有一种注释说 `cache miss, executing`。这意味着`docs`正在运行它的linting。
2. `2 cached, 3 total` 出现在底部。

这意味着**我们之前任务的结果仍然被缓存着**。只有`docs`里面的`lint`脚本才真正运行-这再次加快了事情的速度。要了解更多，请查看我们的[缓存文档](https://turbo.build/repo/docs/core-concepts/caching)。

### 5. Building with Turborepo

让我们尝试运行 `build` 脚本:

### npm
```bash
  npm run build
```

### yarn
```bash
  yarn build
```

### pnpm
```bash
  pnpm run build 
```

你会看到与我们运行lint脚本时类似的输出。只有 `apps/docs` 和 `apps/web` 在它们的 `package.json` 中指定了一个`build`脚本，所以只运行这些脚本。

看一下`turbo.json`中的`build`部分。那里有一些有趣的配置。

```json 
filename="turbo.json"
{
  "pipeline": {
    "build": {
      "outputs": [".next/**"]
    }
  }
}
```

你会注意到有些`输出`已经被指定。声明输出将意味着当`turbo`完成运行你的任务时，它将把你指定的输出保存在它的缓存中。

`apps/docs`和 `apps/web` 都是 Next.js 应用程序，它们会将构建结果输出到 `./.next` 文件夹。

让我们试一试。删除`apps/docs/.next` build文件夹。

再次运行`build`脚本。你会发现:

1. 我们达到了`FULL TURBO`-构建在`100ms`内完成。
2. `.next` 文件夹重新出现了!

Turborepo cached the result of our previous build. When we ran the `build` command again, it restored the entire `.next/**` folder from the cache. To learn more, check out our docs on [cache outputs](/repo/docs/core-concepts/caching#configuring-cache-outputs).

Turborepo 缓存了我们之前构建的结果。当我们再次运行`build`命令时，它从缓存中恢复了整个`.next/**`文件夹。想了解更多，请查看我们关于[缓存输出](https://turbo.build/repo/docs/core-concepts/caching#configuring-cache-outputs)的文档。

### 6. 运行 dev 脚本

现在，让我们尝试运行 `dev`.

### npm
```bash
  npm run dev
```

### yarn
```bash
  yarn dev
```

### pnpm
```bash
  pnpm run dev 
```

你会注意到终端的一些信息:

1. 只有两个脚本会被执行 - `docs:dev` 和 `web:dev`。这是仅有的两个指定 `dev` 的工作空间。
2. 两个`dev`脚本同时运行，在`3000`和`3001`端口启动你的Next.js应用。在终端，你会看到cache bypass，强制执行
3. 在终端，你会看到`缓存旁路，强制执行`。

试着从脚本中退出，然后重新运行。你会注意到我们并没有`FULL TURBO`。这是为什么呢？

看一下 `turbo.json`:

```json 
filename="turbo.json"
{
  "pipeline": {
    "dev": {
      "cache": false
    }
  }
}
```

在`dev`中，我们指定了 `"cache": false`。这意味着我们告诉 Turborepo _不_ 要缓存 `dev` 脚本的结果。`dev` 运行一个持久的 dev 服务器，不产生任何输出，所以缓存没有意义。在我们的文档中了解更多关于[关闭缓存](https://turbo.build/repo/docs/core-concepts/caching#turn-off-caching)的信息。

#### 一次只在一个工作区运行`dev` //Running `dev` on only one workspace at a time

默认情况下，`turbo run dev`会同时在所有工作区运行`dev`。但有时，我们可能只想选择一个工作区。

为了处理这个问题，我们可以在我们的命令中添加一个`--filter`标志。这个`--filter`会传递给`turbo` CLI。

### npm
```bash
  npm run dev --filter docs (源文档是 `-- --filter docs`)
```

### yarn
```bash
  yarn dev --filter docs 
```

### pnpm
```bash
  pnpm run dev --filter docs  
```

你会注意到，现在它只运行`docs:dev`。从我们的文档中了解更多关于[过滤工作空间](https://turbo.build/repo/docs/core-concepts/monorepos/filtering)的信息。

### 摘要

干得好!你已经了解了所有关于你的新monorepo，以及Turborepo是如何让你的任务处理得更简单的。

#### 接下来的步骤

- 需要添加更多任务？了解更多关于使用[管道]的信息
- 想加快CI的速度？设置[远程缓存](https://turbo.build/repo/docs/core-concepts/remote-caching)吧。
- 想要一些灵感吗？看看我们的[例子](https://github.com/vercel/turbo/tree/main/examples)目录吧
