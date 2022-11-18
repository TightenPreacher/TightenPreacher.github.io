
# 在Monorepo测试 //Testing in a Monorepo

与linting和构建一样，测试也是生产就绪的monorepo的一个重要部分。无论你使用的是端到端测试还是单元测试套件，将它们与Turborepo集成都会带来巨大的速度提升。

## 与测试运行器一起工作 //Working with test runners

假设我们有一个monorepo，它看起来像这样:

```
├── apps
│   └── web
│       └── package.json
└── packages
    └── shared
        └── package.json
```

Both `apps/web` and `packages/shared` have their own test suite. Their `package.json` files look like this:

### Jest
```json 
filename="apps/web/package.json"
  {
    "scripts": {
      "test": "jest"
    }
  }
```

### Vitest
```json 
filename="apps/web/package.json"
  {
    "scripts": {
      "test": "vitest run"
    }
  }
```

在根目录 `turbo.json`里面，我们建议在你的[管道](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)中设置一个`测试`任务:

```json 
filename="turbo.json"
{
  "pipeline": {
    "test": {
      "outputs": []
    }
  }
}
```

现在，在你的根目录`package.json`中，添加一个`test`脚本:

```json 
filename="package.json"
{
  "scripts": {
    "test": "turbo run test"
  }
}
```

现在，你可以使用你的软件包管理器运行`测试`，让Turborepo测试整个版本库。

由于Turborepo的[缓存功能](https://turbo.build/repo/docs/core-concepts/caching)，这也意味着只有改变了文件的版本库才会被测试 - 从而节省了大量的时间。

## 在观察模式下运行测试 //Running tests in watch mode

当你正常运行你的测试套件时，它会完成并输出到`stdout`。这意味着你可以用Turborepo[缓存它](https://turbo.build/repo/docs/core-concepts/caching)。

但是当你在观察模式下运行你的测试时，这个过程永远不会退出。这使得观察任务更像一个[开发任务](https://turbo.build/repo/docs/handbook/dev)。

由于这种差异，我们建议指定**两个独立的Turborepo任务**：一个用于运行你的测试，另一个用于在观察模式下运行它们。

下面是一个例子:


### Jest
```json 
filename="apps/web/package.json"
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

### Vitest
```json 
filename="apps/web/package.json"
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest"
  }
}
```

```json 
filename="turbo.json"
{
  "pipeline": {
    "test": {
      "outputs": []
    },
    "test:watch": {
      "cache": false
    }
  }
}
```

```json 
filename="package.json"
{
  "scripts": {
    "test": "turbo run test",
    "test:watch": "turbo run test:watch"
  }
}
```
