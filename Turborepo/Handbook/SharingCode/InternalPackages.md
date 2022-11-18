
# 内部包 //Internal Packages

内部包是只在你的monorepo _内_ 使用的[包](https://turbo.build/repo/docs/handbook/sharing-code)。它们对于在闭源单体的应用程序之间共享代码非常有用。

内部包可以快速创建，而且如果你最终想把它们发布到`npm`上，可以把它们变成[外部包](https://turbo.build/repo/docs/handbook/publishing-packages)。

## 什么使一个包成为 _内部_ 的？ //What makes a package _internal?_

外部包在把它们放在包注册表上之前**通过捆绑器运行它们的文件**。这意味着它们需要 _大量_ 的工具来处理。

- **Bundlers**: 构建软件包
- **Versioning**: 帮助进行版本管理和发布
- **Publishing**: 发布软件包

如果你想在本地使用这些文件，你还需要:

- **Dev scripts**: 用于在文件改变时在本地捆绑软件包

因为内部软件包没有发布，我们可以跳过 _所有_ 这些步骤。我们不需要自己捆绑我们的包，而是让**导入包的应用程序为我们捆绑它**。

这听起来很复杂，但设置起来却非常容易。

## 我们的第一个内部包 //Our first internal package

我们将在我们的monorepo中创建一个共享的`math-helpers`包

### 1. 创建你的monorepo //Create your monorepo

如果你没有一个现有的monorepo，请[使用我们的指南](https://turbo.build/repo/docs/getting-started/create-new)创建一个。

### 2. 创建一个新的包 //Create a new package

在`/packages`中，创建一个名为`math-helpers`的新文件夹。

```bash
mkdir packages/math-helpers
```

创建 `package.json`:

```json 
filename="packages/math-helpers/package.json"
{
  "name": "math-helpers",
  "dependencies": {
    // Use whatever version of TypeScript you're using
    "typescript": "latest"
  }
}
```

创建一个`src`文件夹，并在`packages/math-helpers/src/index.ts`添加一个TypeScript文件。

```ts 
filename="packages/math-helpers/src/index.ts"
export const add = (a: number, b: number) => {
  return a + b;
};

export const subtract = (a: number, b: number) => {
  return a - b;
};
```

你还需要在`packages/math-helpers/tsconfig.json`添加一个`tsconfig.json`:

```json 
filename="packages/math-helpers/tsconfig.json"
{
  "compilerOptions": {
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "moduleResolution": "node",
    "preserveWatchOutput": true,
    "skipLibCheck": true,
    "noEmit": true,
    "strict": true
  },
  "exclude": ["node_modules"]
}
```

很好!我们现在已经得到了我们的内部包所需要的所有文件。

### 3. 导入软件包 //Import the package

我们现在要导入这个包，看看会发生什么。进入你的一个应用程序，将`math-helpers`添加到其package.json的依赖项中:

### npm
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "math-helpers": "*"
  }
}
```

### yarn
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "math-helpers": "*"
  }
}
```

### pnpm
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "math-helpers": "workspace:*"
  }
}
```

[从根部安装所有的软件包](https://turbo.build/repo/docs/handbook/package-installation)，以确保依赖性的工作。

现在，在你的应用程序的一个源文件中添加一个从`math-helpers`导入的文件

```diff
+ import { add } from "math-helpers";

+ add(1, 2);
```

你可能会看到一个错误!

```
Cannot find module 'math-helpers' or its corresponding type declarations.
```

这是因为我们错过了一个步骤。我们还没有告诉我们的`math-helpers/package.json`，我们包的入口点是什么。

### 4. 修复`main`和`types` //Fix `main` and `types`

回到`packages/math-helpers/package.json`中，添加两个字段，`main`和`types`:

```json 
filename="packages/math-helpers/package.json"
{
  "name": "math-helpers",
  "main": "src/index.ts",
  "types": "src/index.ts",
  "dependencies": {
    "typescript": "latest"
  }
}
```

Now, anything that imports our `math-helpers` module will be pointed directly towards the `src/index.ts` file - _that's_ the file that they will import.
现在，任何导入我们的 `math-helpers` 模块的东西都会被直接指向 `src/index.ts` 文件-_这_ 就是他们要导入的文件。

回到 `apps/web/pages/index.tsx`。这个错误应该已经消失了

### 5. 尝试运行该应用程序 //Try running the app

现在，试着运行该应用的开发脚本。在默认的turborepo中，这将是很简单的:

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

### Next.js
当它开始运行时，你可能会在你的网络浏览器中看到一个错误:

```
  ../../packages/math-helpers/src/index.ts
  Module parse failed: Unexpected token (1:21)
  You may need an appropriate loader to handle this file type,
  currently no loaders are configured to process this file.
  See https://webpack.js.org/concepts#loaders

  > export const add = (a: number, b: number) => {
  |   return a + b;
  | };
```

当你试图将一个未捆绑的文件导入Next.js应用程序时，就会出现这种情况。

解决方法很简单-我们需要告诉Next.js将它导入的某些包中的文件捆绑起来。

### Vite
因为`vite`默认是转置模块的，所以不需要更多的设置!跳到第7步。

### 6. 配置你的应用程序 //Configuring your app

### Next.js
我们可以用[`next-transpile-modules`](https://www.npmjs.com/package/next-transpile-modules)来做。

将 `next-transpile-modules` 安装到你的应用程序。如需复习，请参阅我们的[软件包安装](https://turbo.build/repo/docs/handbook/package-installation)文档。

在你的应用程序的`next.config.js`内，添加`next-transpile-modules`:

```ts 
filename="apps/web/next.config.js"
const withTM = require("next-transpile-modules")([
  // Add "math-helpers" to this array:
  "math-helpers",
]);

module.exports = withTM({
  // Any additional config for next goes in here
});
```

重新启动你的开发脚本，并进入浏览器。

**错误已经消失了!**

### Vite
不需要配置!

### 7. 摘要 //Summary

我们现在可以在我们的`math-helpers`包中添加任何数量的代码，并在我们的monorepo中的任何应用程序中使用它。我们甚至不需要构建我们的包--它就能工作。

这种模式对于创建可以在团队之间轻松共享的代码片断是非常强大的。

### 快速参考 //Quick Reference

#### 快速参考 - 创建一个新的内部包 //Quick reference - creating a new internal package

1. 在 `packages/<folder>` 中创建一个新的文件夹
2. 添加`package.json`，其`name`和`types`指向`src/index.ts`（或`src/index.tsx`）。
3. 添加`src/index.tsx`，至少有一个命名的出口
4. 从根目录[安装你的软件包](https://turbo.build/repo/docs/handbook/package-installation)

#### 快速参考-导入一个内部包 //Quick reference - importing an internal package

1. 确保你正在[正确地导入它](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages#3-import-the-package)
2. 确保你已经[配置了你的应用程序来转译它](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages#6-configuring-your-app)
