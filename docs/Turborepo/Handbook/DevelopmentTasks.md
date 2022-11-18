
# Monorepo中的开发任务 //Development tasks in a Monorepo

绝大多数的开发工作流程都是这样的:

1. 打开一个资源库
2. 在他们开发时运行一个`dev`任务
3. 在一天结束时，关闭`dev`任务并关闭版本库

`dev`很可能是你的版本库中最频繁运行的任务，所以把它做好很重要。

## `dev`任务的类型 //Types of `dev` tasks

`dev`任务有多种形式和规模:

1. 为一个网络应用程序运行一个本地开发服务器
2. 运行[`nodemon`](https://www.npmjs.com/package/nodemon)，在每次代码改变时重新运行后端进程
3. 在`--watch`模式下运行测试

## 使用Turborepo进行设置 //Setup with Turborepo

你应该在你的`turbo.json`中这样指定你的`dev`任务。

```json 
filename="turbo.json"
{
  "pipeline": {
    "dev": {
      "cache": false
    }
  }
}
```

由于`dev`任务不产生输出，所以`输出`是空的。`dev`任务也很独特，你很少想[缓存](https://turbo.build/repo/docs/core-concepts/caching)它们，所以我们把缓存设置为false。

### 设置package.json //Setting up `package.json`

你也应该在你的根目录`package.json`中提供一个`dev`任务

```json 
filename="package.json"
{
  "scripts": {
    "dev": "turbo run dev"
  }
}
```

这使开发人员能够直接从他们的正常任务运行器中运行该任务。

## 在`dev`_前_ 运行任务 //Running tasks _before_ `dev`

在某些工作流程中，你想在运行`dev`任务 _之前_ 运行任务。例如，生成代码或运行`db:migrate`任务。

在这些情况下，使用 [dependsOn](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks#in-the-same-workspace) 来表示任何 `codegen` 或 `db:migrate` 任务应该在 `dev` 运行 _之前_ 运行。

```json 
filename="turbo.json"
{
  "pipeline": {
    "dev": {
      "dependsOn": ["codegen", "db:migrate"],
      "cache": false
    },
    "codegen": {
      "outputs": ["./codegen-outputs/**"]
    },
    "db:migrate": {
      "cache": false
    }
  }
}
```

然后，在你的应用程序的 `package.json`:

```json 
filename="apps/web/package.json"
{
  "scripts": {
    // For example, starting the Next.js dev server
    "dev": "next",
    // For example, running a custom code generation task
    "codegen": "node ./my-codegen-script.js",
    // For example, using Prisma
    "db:migrate": "prisma db push"
  }
}
```

这意味着你的`dev`任务的用户 _不需要担心codegen或迁移他们的数据库_-在他们的开发服务器开始之前就已经为他们处理了。

## 只在某些工作区运行`dev`任务 //Running `dev` only in certain workspaces

要想只在某些工作区运行`dev`任务，你应该使用[`--filter`语法](https://turbo.build/repo/docs/core-concepts/monorepos/filtering)。比如说:

```bash
turbo run dev --filter docs
```

只会在名为`docs`的工作区运行`dev`。

## 使用环境变量 //Using environment variables

在开发过程中，你经常需要使用环境变量。这些变量可以让你自定义你的程序的行为-例如，在开发和生产中指向一个不同的`DATABASE_URL`。

我们推荐使用一个叫做[`dotenv-cli`](https://www.npmjs.com/package/dotenv-cli)的库来解决这个问题。

我们希望每个开发人员都能有一个很好的使用Turbo的体验。下面记录的方法并不符合这些标准。

我们正在为这个问题开发一个一流的解决方案 - 但在你等待的时候，这里有一个次好的解决方案。

### Tutorial

1. 在你的根工作区[root workspace](https://turbo.build/repo/docs/handbook/what-is-a-monorepo#the-root-workspace)安装`dotenv-cli`Install:

### npm
```bash
  # Installs dotenv-cli in the root workspace
  npm add dotenv-cli
```

### yarn
```bash
  # Installs dotenv-cli in the root workspace
  yarn add dotenv-cli --ignore-workspace-root-check
```

### pnpm
```bash
  # Installs dotenv-cli in the root workspace
  pnpm add dotenv-cli --ignore-workspace-root-check
```

1. 在你的根工作区添加一个.env文件:

```diff
  ├── apps/
  ├── packages/
+ ├── .env
  ├── package.json
  └── turbo.json
```

添加任何你需要注入的环境变量:

```txt 
filename=".env"
DATABASE_URL=my-database-url
```

3. 在你的根目录`package.json`中，添加一个`dev`脚本。用`dotenv`和`--`参数分隔符作为它的前缀:

```json
{
  "scripts": {
    "dev": "dotenv -- turbo run dev"
  }
}
```

这将在运行`turbo run dev`之前从`.env`中提取环境变量。

4. 现在，你可以运行你的dev脚本:


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
   pnpm dev  
```

而你的环境变量将被填充!在Node.js中，这些都可以在`process.env.DATABASE_URL`上找到。

如果你使用环境变量来构建你的应用程序，你也应该把你的[环境变量添加到](/repo/docs/core-concepts/caching#altering-caching-based-on-environment-variables)你的`turbo.json`中。