
# 在monorepo 的 ESLint //ESLint in a monorepo

## 共享配置 //Sharing config

在工作区之间共享ESLint配置，可以使你的所有工作区更加一致，从而促进生产力。

让我们想象一下这样的一个monorepo:

```
apps
├─ docs
│  ├─ package.json
│  └─ .eslintrc.js
└─ web
   ├─ package.json
   └─ .eslintrc.js
packages
└─ eslint-config-custom
   ├─ index.js
   └─ package.json
```

我们有一个名为`eslint-config-custom`的包，以及两个应用程序，每个都有自己的`.eslintrc.js`。

### 我们的`eslint-config-custom`包 //Our `eslint-config-custom` package

我们的`eslint-config-custom`文件只包含一个文件，`index.js`。它看起来像这样。

```js 
filename="packages/eslint-config-custom/index.js"
module.exports = {
  extends: ["next", "turbo", "prettier"],
  rules: {
    "@next/next/no-html-link-for-pages": "off",
    "react/jsx-key": "off",
  },
};
```

这是一个典型的ESLint配置，没有什么花哨的。

`package.json`看起来像这样:

```json 
filename="packages/eslint-config-custom/package.json"
{
  "name": "eslint-config-custom",
  "main": "index.js",
  "dependencies": {
    "eslint": "latest",
    "eslint-config-next": "latest",
    "eslint-config-prettier": "latest",
    "eslint-plugin-react": "latest",
    "eslint-config-turbo": "latest"
  }
}
```

这里有两件事值得注意。首先，`main`字段指向`index.js`。这允许文件轻松[导入这个配置](https://turbo.build/repo/docs/handbook/sharing-code#anatomy-of-a-package)。

第二，ESLint的依赖关系都在这里列出。这很有用-这意味着我们不需要在导入`eslint-config-custom`的应用程序中重新指定依赖关系。

### 如何使用`eslint-config-custom`包 //How to use the `eslint-config-custom` package

在我们的`web`应用程序中，我们首先需要添加`eslint-config-custom`作为一个依赖项。

### npm
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "eslint-config-custom": "*"
  }
}
```

### yarn
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "eslint-config-custom": "*"
  }
}
```

### pnpm
```jsonc 
filename="apps/web/package.json"
{
  "dependencies": {
    "eslint-config-custom": "workspace:*"
  }
}
```

然后我们可以像这样导入配置:

```js 
filename="apps/web/.eslintrc.js"
module.exports = {
  root: true,
  extends: ["custom"],
};
```

通过在我们的`extends`数组中加入`custom`，我们告诉ESLint寻找一个叫做`eslint-config-custom`的包-并找到我们的工作空间。

### 总结

当你用`npx create-turbo@latest`[创建一个新的monorepo](https://turbo.build/repo/docs/getting-started/create-new)时，这个设置是默认的。你也可以看一下我们的[基本例子](https://github.com/vercel/turbo/tree/main/examples/basic)，看看工作版本。

## 设置一个`lint`任务 //Setting up a `lint` task

我们建议遵循[`基础`](https://turbo.build/repo/docs/handbook/linting#running-tasks)部分的设置，但有一个改动。

每个`package.json`脚本应该是这样的

```json 
filename="packages/*/package.json"
{
  "scripts": {
    "lint": "eslint"
  }
}
```
