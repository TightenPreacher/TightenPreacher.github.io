
# 在monorepo 的 TypeScript //TypeScript in a monorepo

你可以通过两种方式之一在monorepo中使用TypeScript-作为一个linter，或者作为一个构建工具。

在这一节中，我们将讨论TypeScript作为linter的作用。这是当你阻止TypeScript发射文件（用[`noEmit`](https://www.typescriptlang.org/tsconfig/noEmit.html))），而 _只是_ 用它来检查你的源代码的类型。

## 共享`tsconfig.json` //Sharing `tsconfig.json`

我们可以通过一个巧妙的解决方案在我们的存储库中共享TypeScript配置文件。我们可以把我们的 _基础_ `tsconfig.json`文件放在一个工作区，并从我们应用程序中的`tsconfig.json`文件中`扩展`它们。

让我们想象一下这样的工作空间:

```
apps
├─ docs
│  ├─ package.json
│  ├─ tsconfig.json
├─ web
│  ├─ package.json
│  ├─ tsconfig.json
packages
├─ tsconfig
│  ├─ base.json
│  ├─ nextjs.json
│  ├─ package.json
│  ├─ react-library.json
```

### 我们的`tsconfig`包 //Our `tsconfig` package

在`packages/tsconfig`里面，我们有几个`json`文件，代表了你可能想要配置TypeScript的不同方式。它们看起来都是这样的:

```json 
filename="packages/tsconfig/base.json"
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "display": "Default",
  "compilerOptions": {
    "composite": false,
    "declaration": true,
    "declarationMap": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "inlineSources": false,
    "isolatedModules": true,
    "moduleResolution": "node",
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "preserveWatchOutput": true,
    "skipLibCheck": true,
    "strict": true
  },
  "exclude": ["node_modules"]
}
```

在`package.json`中，我们简单地将我们的包命名为:

```json 
filename="packages/tsconfig/package.json"
{
  "name": "tsconfig"
}
```

存储库中的其他`json`文件可以通过一个简单的导入访问:

```ts
import baseJson from "tsconfig/base.json";
import nextjsJson from "tsconfig/nextjs.json";
import reactLibraryJson from "tsconfig/react-library.json";
```

这让我们可以为不同类型的项目输出不同的配置设置。

### 如何使用`tsconfig`包 //How to use the `tsconfig` package

每个使用我们共享的`tsconfig`的应用程序/软件包必须首先将其指定为一个依赖项:

### npm
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "tsconfig": "*"
  }
}
```

### yarn
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "tsconfig": "*"
  }
}
```

### pnpm
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "tsconfig": "workspace:*"
  }
}
```

然后，他们可以**在自己的`tsconfig.json`中扩展它**:

```json 
filename="apps/web/tsconfig.json"
{
  // We extend it from here!
  "extends": "tsconfig/nextjs.json",

  // You can specify your own include/exclude
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

### 总结

当你用`npx create-turbo@latest`[创建一个新的monorepo](https://turbo.build/repo/docs/getting-started/create-new)时，这个设置是默认提供的。你也可以看一下[我们的基本例子](https://github.com/vercel/turbo/tree/main/examples/basic)，看看工作版本。


## 运行任务 //Running tasks

我们建议按照[`基础`](/repo/docs/handbook/linting#running-tasks)部分的设置进行。
