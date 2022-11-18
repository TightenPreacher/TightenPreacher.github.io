
# 在Turborepo中使用Prisma //Using Prisma with Turborepo

[Prisma](https://www.prisma.io/)是一个非常流行的ORM，具有自动迁移、类型安全和集成工具的功能。与Turborepo一起使用它可以减少你生成代码的时间，并轻松确保你生成的Prisma代码始终是最新的。

## 指南 //Guide  

本指南告诉你如何:

1. 在monorepo中设置Prisma
2. 处理迁移和代码生成脚本
3. 用 Turborepo 缓存这些脚本
4. 确保每当运行`dev`或`build`时都会运行这些脚本

如果你已经在数据库中设置了Prisma，你可以跳到[第四步](https://turbo.build/repo/docs/handbook/prisma#4-setting-up-the-scripts)

### 1. 创建你的Monorepo //Create your Monorepo

如果你没有一个现有的项目，使用我们的[快速入门](https://turbo.build/repo/docs/getting-started/create-new)来创建一个新的monorepo。

### 2. 添加一个新的`数据库`包 //Add a new `database` package

在packages中创建一个名为`database`的新文件夹，里面有一个`package.json`:

```json 
filename="packages/database/package.json"
{
  "name": "database",
  "dependencies": {
    "@prisma/client": "latest"
  },
  "devDependencies": {
    // Replace "latest" with the latest version
    "prisma": "latest"
  }
}
```

如果你使用的是`pnpm`，你应该在根部添加一个名为`.npmrc`的文件

```txt 
filename=".npmrc"
public-hoist-pattern[]=*prisma*
```

运行你的软件包管理器的安装步骤来安装新的依赖项

### 3. 运行 `prisma init` //Run `prisma init`

`cd` 到 `packages/database`:

```bash
cd packages/database
```

运行 `npx prisma init`.

这应该在`packages/database`中创建几个文件

```
prisma/schema.prisma
.gitignore
.env
```

- `schema.prisma` 是你的[Prisma模式](https://www.prisma.io/docs/concepts/components/prisma-schema)所在的地方。在这里，你将能够修改你的数据库的形状。
- `.gitignore`  将一些被忽略的文件添加到 git 中
- `.env` 让你手动指定你的`DATABASE_URL`，用于prisma.

在这一点上，你应该参考Prisma文档，将你的[数据库连接到Prisma](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases/connect-your-database-typescript-postgres)。

一旦你连接了一个数据库，你就可以继续前进了

### 4. 设置脚本 //Setting up the scripts

让我们在`packages/database`里面的`package.json`中添加一些脚本:

```json 
filename="packages/database/package.json"
{
  "scripts": {
    "db:generate": "prisma generate",
    "db:push": "prisma db push --skip-generate"
  }
}
```

让我们也把这些脚本添加到根目录下的`turbo.json`中:

```json 
filename="turbo.json"
{
  "pipeline": {
    "db:generate": {
      "cache": false
    },
    "db:push": {
      "cache": false
    }
  }
}
```

现在我们可以从我们仓库的根部运行`turbo db:push db:generate`，以自动迁移我们的数据库并生成我们的类型安全的Prisma客户端。

我们在 `db:push` 上使用 `--skip-generate` 标志，以确保它在迁移数据库后不会自动运行 `Prisma generate`。在使用Turborepo时，由于我们自动将这些任务并行化，所以最终会更快。

### 5. 导出你的客户端 //Exporting your client

接下来，我们需要导出`@prisma/client`，这样我们就可以在我们的应用程序中使用它。让我们在`packages/database`中添加一个新文件:

```ts 
filename="packages/database/index.ts"
export * from '@prisma/client';
```

Following the [internal packages pattern](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages), we'll also need to add `index.ts` to `main` and `types` inside `packages/database/package.json`.

按照[内部包的模式](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages)，我们还需要将`index.ts`添加到`main`和`types`在`packages/database/package.json`里面。

```json 
filename="packages/database/package.json"
{
  "main": "./index.ts",
  "types": "./index.ts"
}
```

#### 导入`database`  //Importing `database`

现在让我们把我们的数据库包导入我们的一个应用程序中进行测试。假设你在`apps/web`有一个应用程序。将依赖关系添加到`apps/web/package.json`中:

### npm
```json 
  filename="apps/web/package.json"
  {
    "dependencies": {
      "database": "*"
    }
  }
```

### yarn
```json 
filename="apps/web/package.json"
  {
    "dependencies": {
      "database": "*"
    }
  }
```

### pnpm
```json 
filename="apps/web/package.json"
  {
    "dependencies": {
      "database": "workspace:*"
    }
  }
```

运行你的软件包管理器的安装命令。

现在你可以在你的应用程序的任何地方从`database`中导入`PrismaClient`:

```ts
import { PrismaClient } from 'database'

const client = new PrismaClient();
```

你可能还需要在你的应用程序中做一些配置，以允许它运行一个内部包。请查看我们的[内部软件包文档](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages#6-configuring-your-app)以获得更多信息。

### 6. 摸清脚本 //Figuring out the scripts

我们现在处于一个相当好的位置。我们有一个可重复使用的`database`模块，可以导入到我们的任何应用程序中。我们已经有了一个`turbo db:push`脚本，我们可以用它来推送我们的变化到数据库。

然而，我们的`db:generate`脚本还没有被优化。它们为我们的`dev`和`build`任务提供了关键的代码。如果一个新的开发者在没有先运行`db:generate`的情况下在一个应用程序上运行`dev`，他们会得到错误。

所以，让我们确保`db:generate`总是在用户运行`dev` _之前_ 运行:

```json 
filename="turbo.json"
{
  "pipeline": {
    "dev": {
      "dependsOn": ["^db:generate"],
      "cache": false
    },
    "build": {
      "dependsOn": ["^db:generate"],
      "outputs": ["your-outputs-here"]
    },
    "db:generate": {
      "cache": false
    }
  }
}
```

查看关于[运行任务](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks)的章节，了解更多关于`^db:generate`语法的信息。

### 7.缓存`prisma generate`的结果 // Caching the results of `prisma generate`

`prisma generate` 将文件输出到文件系统中，通常在 `node_modules` 中。理论上，应该可以用 Turborepo 缓存 `prisma generate` 的输出，以节省几秒钟。

然而，Prisma 在不同的软件包管理器中的表现是不同的。这可能会导致不可预知的结果，在某些情况下可能会导致部署失败。与其记录每种方法的复杂性，我们建议 _不要_ 缓存`prisma generate`的结果。由于`prisma generate`生成通常只需要5-6秒，而且在较大的`schema`文件中往往不会花费更多时间，这似乎是一个很好的权衡。

你可能想自己试验一下。如果你找到了一个你认为可行的解决方案，请随时[添加一个问题](https://github.com/vercel/turbo/issues/new/choose)，我们可以更新这个部分。