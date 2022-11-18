
# 构建你的应用程序 //Building your App

除非你的monorepo只是用来[向npm发布软件包](https://turbo.build/repo/docs/handbook/publishing-packages)，否则它可能至少包含一个应用程序。用Turborepo来协调你的应用程序的构建，可以带来一些非凡的速度。

## 设置构建 //Setting up the build

Turborepo的工作原理是将工作区的任务放在属于它们的地方-每个工作区的`package.json`中。让我们想象一下，你有一个这样的monorepo:

```
├── apps
│   └── web
│       └── package.json
├── package.json
└── turbo.json
```

你的`apps/web/package.json`里面应该有一个`build`脚本
Your `apps/web/package.json` should have a `build` script inside:

### Next.js
```json 
filename="apps/web/package.json"
{
  "scripts": {
    "build": "next build"
  }
}
```

### Vite
```json 
filename="apps/web/package.json"
{
  "scripts": {
    "build": "vite build"
  }
}
```

在`turbo.json`里面，你可以把`build`添加到管道里。

### Next.js
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

### Vite
```json 
filename="turbo.json"
{
  "pipeline": {
    "build": {
      "outputs": ["dist/**"]
    }
  }
}
```

我们配置`输出`，这样就可以启用[缓存](https://turbo.build/repo/docs/core-concepts/caching)-这是Turborepo的一个极其强大的功能，可以跳过以前做过的任务。

现在，在你的根目录`package.json`中添加一个脚本:

```json 
filename="package.json"
{
  "scripts": {
    "build": "turbo run build"
  }
}
```

这意味着从根目录下使用软件包管理器运行`build`，将构建软件库中的所有应用程序。多亏了Turborepo的任务缓存，你可以获得极快的构建时间。