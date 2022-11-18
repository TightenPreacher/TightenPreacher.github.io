
# 在monorepo中共享代码 //Sharing Code in a monorepo

monorepos让你在不同的应用程序之间分享代码，而没有摩擦。要做到这一点，你将建立**packages**，在你的应用程序之间共享代码。

## 什么是package //What is a package?

在谈到monorepos时，"package"这个词有双重含义。它可以指的是其中的任何一种:

1. 一组你从注册表下载到你的`node_modules`的文件，通过像`npm`这样的包管理器
2. 一个包含代码的工作空间，可以在应用程序之间共享-按照惯例，通常在`/packages`里面

这种双重含义可能会让刚开始接触monorepo的人感到非常困惑。你可能对[软件包的安装](https://turbo.build/repo/docs/handbook/package-installation)非常熟悉，但对[工作空间](https://turbo.build/repo/docs/handbook/workspaces)却不那么熟悉。

事实上，它们是非常相似的。一个包只是一段共享代码。除了 _已安装的软件包_ 在你的`node_modules`中，而 _本地软件包_ 在工作空间中-可能在你的`/packages`文件夹中。

## package //Anatomy of a package

每个包都包含一个`package.json`。你可能很熟悉使用这些包来管理你应用程序中的依赖关系和脚本。

然而，你可能没有注意到之前的`main`和`name`字段:

```jsonc 
filename="packages/my-lib/package.json"
{
  // The name of your package
  "name": "my-lib",

  // When this package is used, this file is actually
  // the thing that gets imported
  "main": "./index.js"
}
```

这两个字段对于决定**这个包在被导入时的行为方式**都很重要。例如，如果`index.js`有一些出口

```js 
filename="packages/my-lib/index.js"
export const myFunc = () => {
  console.log("Hello!");
};
```

我们将这个文件导入我们的一个应用程序中:

```ts 
filename="apps/web/pages/index.jsx"
import { myFunc } from "my-lib";

myFunc(); // Hello!
```

然后我们就可以在我们的应用程序中使用`my-lib`文件夹内的代码。

总而言之，**每个包都必须在其`package.json`中声明一个`name`和一个`main`文件**。

`package.json`中的包解析是一个非常复杂的话题，我们不能在这里做公正的描述。`package.json`中的其他字段可能优先于`main`，这取决于软件包的导入方式。

请查看[npm文档](https://docs.npmjs.com/cli/v8/configuring-npm/package-json/#main)中的指南。

对于我们的目的，使用`main`就足够了。

## 接下来的步骤 //Next steps

我们将介绍两种风格的包-**内部**包和**外部**包:

[**内部**包](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages)的目的是只在它们所在的monorepo内使用。它们的设置相对简单，如果你的项目是闭源的，它们将对你最有用。

[**外部**包](https://turbo.build/repo/docs/handbook/publishing-packages)被捆绑起来并被发送到包注册中心。这对设计系统、共享实用程序库或任何开源工作都很有用。然而，它们围绕捆绑、版本和发布引入了更多的复杂性。