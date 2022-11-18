
# 工作空间 //Workspaces

工作区是你的monorepo的构建模块。你添加到monorepo中的每个应用和包都会**在它自己的工作空间中**。

工作区是由你的[软件包管理器](https://turbo.build/repo/docs/handbook/package-installation)管理的，所以请确保你先把它设置好。

## 配置工作空间 //Configuring workspaces

要使用工作空间，你必须首先向你的软件包管理器声明它们的文件系统位置。

我们推荐的一个通用惯例是拥有顶级的`apps/`和`packages/`目录。这不是一个要求-只是一个建议的目录结构。

`apps`文件夹应该包含可启动的应用程序的工作空间，如[Next.js](https://nextjs.org/)或[Svelte](https://svelte.dev/)应用程序。

`packages`文件夹应该包含应用或其他包所使用的包的工作空间。

### npm
将你想配置为`workspaces`的文件夹添加到根目录`package.json`文件的工作空间字段中。这个字段包含一个以globs形式存在的工作空间文件夹的列表:

```json
{
  "name": "my-monorepo",
  "version": "1.0.0",
  "workspaces": [
    "docs",
    "apps/*",
    "packages/*"
  ]
}
```

### yarn
将你想配置为`workspaces`的文件夹添加到根目录`package.json`文件的工作空间字段中。这个字段包含一个以globs形式存在的工作空间文件夹的列表:

```json
{
  "name": "my-monorepo",
  "version": "1.0.0",
  "workspaces": [
    "docs",
    "apps/*",
    "packages/*"
  ]
}
```

### pnpm
将你想配置为工作空间的文件夹添加到存在于你根目录下的 `pnpm-workspace.yaml` 文件中。这个文件包含一个以globs形式存在的工作空间文件夹列表:

```yaml
packages:
  - "docs"
  - "apps/*"
  - "packages/*"
```

```
my-monorepo
├─ docs
├─ apps
│  ├─ api
│  └─ mobile
├─ packages
│  ├─ tsconfig
│  └─ shared-utils
└─ sdk
```

在上面的例子中，`my-monorepo/apps/`和`my-monorepo/packages/`里面的所有目录都是工作空间，my-monorepo/docs目录本身也是一个工作空间。`my-monorepo/sdk/`_不是_ 一个工作空间，因为它没有包含在工作空间配置中。

## 命名工作区 //Naming workspaces

每个工作空间都有一个唯一的名字，在其`package.json`中指定

```json 
filename="packages/shared-utils/package.json"
{
  "name": "shared-utils"
}
```

这个名字是用来:

1. 指定[软件包应该被安装到](https://turbo.build/repo/docs/handbook/package-installation)哪个工作区
2. 在其他工作空间中使用这个工作空间
3. [发布软件包](https://turbo.build/repo/docs/handbook/publishing-packages): 它将以你指定的`名字`发布在npm上

你可以使用npm组织或用户范围来避免与npm上现有的包相冲突。例如，你可以使用`@mycompany/shared-utils`。

## 互相依赖的工作区 //Workspaces which depend on each other

要在另一个工作区中使用一个工作区，你需要用它的名字把它指定为一个依赖关系。

例如，如果我们想让`apps/docs`导入`packages/shared-utils`，我们需要在`apps/docs/package.json`中把`shared-utils`作为一个依赖项。

### npm
```json 
filename="apps/docs/package.json"
{
  "dependencies": {
    "shared-utils": "*"
  }
}
```

### yarn
```json 
filename="apps/docs/package.json"
{
  "dependencies": {
    "shared-utils": "*"
  }
}
```

### pnpm
```json 
filename="apps/docs/package.json"
{
  "dependencies": {
    "shared-utils": "workspace:*"
  }
}
```

这个`*`允许我们引用依赖的最新版本。如果我们的软件包的版本发生变化，它可以使我们不需要修改我们的依赖关系的版本。

就像一个普通的软件包一样，我们需要在之后从根目录下运行`安装`。一旦安装完毕，我们就可以像使用`node_modules`中的任何其他包一样使用工作区。更多信息请参见我们的[共享代码部分](https://turbo.build/repo/docs/handbook/sharing-code)。

## 管理工作空间 //Managing workspaces

在monorepo中，当你从root运行`安装`命令时，会发生一些事情:

1. 检查你所安装的工作空间的依赖性
2. 任何工作空间都被[符号链接](https://en.wikipedia.org/wiki/Symbolic_link)到`node_modules`中，这意味着你可以像普通软件包一样导入它们
3. 其他软件包被下载并安装到`node_modules`中。

这意味着每当你添加/删除工作空间，或者改变它们在文件系统中的位置时，你需要从根目录下重新运行`安装`命令来重新设置你的工作空间。

你**不需要在每次你的源代码在包内发生变化时都重新安装**-只有在你以某种方式改变你的工作空间的位置（或配置）时才需要。

If you run into issues, you may have to delete each `node_modules` folder in your repository and re-run `install` to correct it.
如果你遇到问题，你可能必须删除你的仓库中的每个`node_modules`文件夹，然后重新运行`安装`来纠正它。