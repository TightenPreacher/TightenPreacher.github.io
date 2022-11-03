
# 将Turborepo添加到你现有的项目中 //Add Turborepo to your existing project

Turborepo可以在**任何项目**中使用，以加快`package.json`中脚本的执行速度。


在你安装了`turbo`之后，你就可以从`turbo`而不是软件包管理器中运行你所有的`package.json`任务。

通过正确配置你的`turbo.json`，你会注意到[缓存](https://turbo.build/repo/docs/core-concepts/caching)何帮助你的任务运行得更快。

## 快速入门 //Quickstart

0. **如果你还没有，请创建一个新的应用程序:**

### Next.js

```bash
npx create-next-app@latest
```

### Vite
```bash
npm create vite@latest
```

1. **安装 `turbo`:**

### npm
```bash
  npx create-next-app@latest
```

### yarn
```bash
  yarn add turbo --dev
```

### pnpm
```bash
  pnpm install turbo --save-dev
```

2. **在你的新版本库的基础上添加一个 `turbo.json` 文件:**

### Next.js

```json filename="turbo.json"
  {
    "pipeline": {
      "build": {
        "outputs": [".next/**"]
      },
      "lint": {
        "outputs": []
      }
    }
  }
```

### Vite
```json filename="turbo.json"
  {
    "pipeline": {
      "build": {
        "outputs": ["dist/**"]
      },
      "lint": {
        "outputs": []
      }
    }
  }
```

一些Vite启动器的`package.json`看起来像这样:

```json filename="package.json"
{
  "scripts": {
    "build": "tsc && vite build"
  }
}
```

我们建议将这些内容拆分为`lint`和`build`脚本。

```json filename="package.json"
{
  "scripts": {
    "build": "vite build",
    "lint": "tsc"
  }
}
```

这意味着Turbo可以单独安排它们。

3. **Try running `build` and `lint` with `turbo`:**

### npm
```bash
  npx turbo build lint
```

### yarn
```bash
  yarn turbo build lint
```

### pnpm
```bash
  pnpm turbo build lint
```

这将同时运行`build`和`lint`。

4. **在不对代码做任何修改的情况下，尝试再次运行`build`和`lint`:**

### npm
```bash
  npx turbo build lint
```

### yarn
```bash
  yarn turbo build lint
```

### pnpm
```bash
  pnpm turbo build lint
```

你应该看到像这样的终端输出:

```
 Tasks:    2 successful, 2 total
Cached:    2 cached, 2 total
  Time:    185ms >>> FULL TURBO
```

恭喜 - **你刚刚在200毫秒内完成了构建和lint**.

要了解如何实现这一点，请查看我们的[核心概念文档](https://turbo.build/repo/docs/core-concepts/caching)

5. **尝试用`turbo`运行`dev`:**

### npm
```bash
  npx turbo dev
```

### yarn
```bash
  yarn turbo dev
```

### pnpm
```bash
  pnpm turbo dev
```

你会注意到，你的`dev`脚本启动了。你可以用`turbo`来运行`package.json`中的任何脚本。