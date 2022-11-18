# 疑难解答

## 处理不匹配的软件包版本 //Handling mismatched package versions

随着你的monorepo的增长，你可能最终会在不同的工作区有不同版本的软件包。

例如，`app`可能使用`react@18.0.0`，而`web`可能使用`react@17.0.0`。当你刚刚[从一个多版本的设置中迁移过来](https://turbo.build/repo/docs/handbook/migrating-to-a-monorepo)时，这一点尤其真实。

不同仓库中的错误匹配的依赖关系可能意味着代码的运行会出现意外。例如，React，如果安装了不止一个版本，就会出错。

#### `@manypkg/cli`

我们推荐使用[`@manypkg/cli`](https://www.npmjs.com/package/@manypkg/cli)来处理这个问题-这是一个CLI，可以确保你的依赖关系在你的仓库中匹配。

这里有一个快速的例子。在你的`package.json`的根目录，添加一个`postinstall`脚本。

```json 
filename="package.json"
{
  "scripts": {
    // This will check your dependencies match
    // after each installation
    "postinstall": "manypkg check"
  },
  "dependencies": {
    // Make sure you install @manypkg/cli
    "@manypkg/cli": "latest"
  }
}
```

你也可以运行`manypkg fix`来自动更新整个版本库。