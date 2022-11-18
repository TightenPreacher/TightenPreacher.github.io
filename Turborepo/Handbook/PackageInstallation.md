
# 依赖安装 //Package Installation

一个软件包管理器（如`npm`）为你处理两件事：[管理工作空间](https://turbo.build/repo/docs/handbook/workspaces)和安装软件包

Turborepo与四个软件包管理器兼容:

- [npm](https://docs.npmjs.com/cli/v8/using-npm/workspaces/#description)
- [pnpm](https://pnpm.io/workspaces)
- [Yarn 1](https://classic.yarnpkg.com/lang/en/docs/workspaces/)
- Yarn >=2 (即将推出的文档)

你应该使用你觉得最舒服的那一种 - 但**如果你是一个monorepo的初学者，我们推荐npm**。

If you're **comfortable with monorepos, we recommend pnpm**. It's faster and offers some useful CLI options like `--filter`.
如果你对**monorepos很熟悉，我们建议使用pnpm**。它更快，并提供了一些有用的CLI选项，如`--filter`。

## 安装软件包 //Installing packages

当你第一次克隆或创建你的monorepo时，你将需要:

1. 确保你在你的monorepo的根目录下
2. 运行安装命令:

### npm
```bash 
    npm install
```

### yarn
```bash 
    yarn install
```

### pnpm
```bash 
    pnpm install
```

现在你会看到 `node_modules` 文件夹出现在版本库的根部和每个工作区中。

## 添加/删除/升级软件包 //Adding/removing/upgrading packages

你可以使用你的软件包管理器的内置命令在你的monorepo中添加、删除和升级软件包:

### npm
**在工作区安装一个软件包**
```bash
    npm install <package> --workspace=<workspace>
```

例子:
```bash
    npm install react --workspace=web
``` 

**从工作区删除一个包**
```bash
   npm uninstall <package> --workspace=<workspace>
```

例子:
```bash
    npm uninstall react --workspace=web
```

**在工作区升级一个软件包**
```bash
    npm update <package> --workspace=<workspace>
```

例子:
```bash
    npm update react --workspace=web
```

### yarn
**在工作区安装一个软件包**
```bash
    yarn workspace <workspace> add <package>
```

例子:
```bash
    yarn workspace web add react
``` 

**从工作区删除一个包**
```bash
   yarn workspace <workspace> remove <package>
```

例子:
```bash
   yarn workspace web remove react
```

**在工作区升级一个软件包**
```bash
    yarn workspace <workspace> upgrade <package>
```

例子:
```bash
    yarn workspace web upgrade react
```

### pnpm
**在工作区安装一个软件包**
```bash
    pnpm add <package> --filter <workspace>
```

例子:
```bash
    pnpm add react --filter web
``` 

**从工作区删除一个包**
```bash
   pnpm uninstall <package> --filter <workspace>
```

例子:
```bash
    pnpm uninstall react --filter web
```

**在工作区升级一个软件包**
```bash
    pnpm update <package> --filter <workspace>
```

例子:
```bash
    pnpm update react --filter web
```