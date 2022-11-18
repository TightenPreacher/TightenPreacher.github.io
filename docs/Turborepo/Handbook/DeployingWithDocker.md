
# 使用Docker进行部署 //Deploying with Docker

构建[Docker](https://www.docker.com/)镜像是部署各种应用程序的常见方式。然而，从一个单库中这样做有几个挑战。

## 问题是 //The problem

**TL;DR：**在monorepo中，不相关的变化会使Docker在部署你的应用程序时做不必要的工作。

让我们想象一下，你有一个看起来像这样的monorepo:

```txt
├── apps
│   ├── docs
│   │   ├── server.js
│   │   └── package.json
│   └── web
│       └── package.json
├── package.json
└── package-lock.json
```

你想使用Docker来部署`apps/docs`，所以你创建了一个Docker文件:

```docker 
filename="Dockerfile"
FROM node:16

WORKDIR /usr/src/app

# Copy root package.json and lockfile
COPY package.json ./
COPY package-lock.json ./

# Copy the docs package.json
COPY apps/docs/package.json ./apps/docs/package.json

RUN npm install

# Copy app source
COPY . .

EXPOSE 8080

CMD [ "node", "apps/docs/server.js" ]
```

这将复制根目录`package.json`和根lockfile到docker镜像上。然后，它将安装依赖性，复制应用程序的源代码并启动应用程序。

你还应该创建一个`.dockerignore`文件，以防止node_modules与应用程序的源代码一起被复制进来。

```txt 
filename=".dockerignore"
node_modules
npm-debug.log
```

### 锁文件变化太频繁 //The lockfile changes too often

Docker对于如何部署你的应用程序是非常聪明的。就像Turbo一样，它试图[尽可能地少做工作](https://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/)。

在我们的Dockerfile中，只有当它的镜像中的文件与之前的运行 _不同_ 时，它才会运`行npm install`。如果没有，它就会恢复它之前的`node_modules`目录。


这意味着，只要`package.json`、`apps/docs/package.json`或`package-lock.json`发生变化，docker镜像就会运行`npm install`。

这听起来不错-直到我们意识到一些问题。`package-lock.json`对monorepo来说是 _全局_ 的。这意味着，**如果我们在`apps/web`中安装一个新的包，我们将导致`apps/docs`重新部署**。

在一个大型的monorepo中，这可能会导致大量的时间损失，因为任何对monorepo的锁文件的改变都会导致数十或数百次的部署。

## 解决办法 //The solution

解决方法是对Docker文件的输入进行修剪，只保留严格必要的内容。Turborepo提供了一个简单的解决方案-`turbo prune`。

```bash
turbo prune --scope="docs" --docker
```

运行这个命令可以在`./out`目录下创建一个**修剪过的monorepo版本**。它只包括`docs`依赖的工作空间。

最重要的是，它还**修剪了lockfile**，所以只有相关的`node_modules`会被下载。

### `--docker`标志 //The `--docker` flag

默认情况下，`turbo prune会`将所有相关文件放在`./out`里面。但为了优化Docker的缓存，我们最好分两个阶段将文件复制过来。

首先，我们希望只复制安装软件包所需的文件。当运行`--docker`时，你会在`./out/json`中找到这个文件。

```txt
out
├── json
│   └── apps
│       └── docs
│           └── package.json
├── full
│   └── apps
│       └── docs
│           ├── server.js
│           └── package.json
└── package-lock.json
```

之后，你可以复制`./out/full`中的文件来添加源文件。

以这种方式拆分**依赖关系**和**源文件**，让我们**只在依赖关系发生变化时才运行`npm install`**-给我们带来更大的速度提升。

如果没有`--docker`，所有被修剪的文件都被放在`./out`里面。

### 例子 //Example

我们详细的[`with-docker`例子](https://github.com/vercel/turbo/tree/main/examples/with-docker)深入介绍了如何充分利用`prune`的潜力。这里是Docker文件，为方便起见复制过来。

```docker
FROM node:alpine AS builder
RUN apk add --no-cache libc6-compat
RUN apk update
# Set working directory
WORKDIR /app
RUN yarn global add turbo
COPY . .
RUN turbo prune --scope=web --docker

# Add lockfile and package.json's of isolated subworkspace
FROM node:alpine AS installer
RUN apk add --no-cache libc6-compat
RUN apk update
WORKDIR /app

# First install the dependencies (as they change less often)
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/yarn.lock ./yarn.lock
RUN yarn install

# Build the project
COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json
RUN yarn turbo run build --filter=web...

FROM node:alpine AS runner
WORKDIR /app

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=installer /app/apps/web/next.config.js .
COPY --from=installer /app/apps/web/package.json .

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static

CMD node apps/web/server.js
```