# Turborepo 快速入门

Turborepo是一个为 **JavaScript和TypeScript代码库优化的智能构建系统**。

你的代码库的任务--如 `lint`、 `构建`和 `测试`--**不能像它们那样快速运行**。Turborepo 使用[缓存](https://turbo.build/repo/docs/core-concepts/caching)来提升你的本地设置并加快你的 CI。

Turborepo被设计成可以**逐步采用**，所以你可以在几分钟内将其添加到大多数代码库中。

**[添加到现有项目中](GettingStarted/AddtoExistingProject.md)** / 
**[创建一个新的项目](GettingStarted/CreateANewMonorepo.md)**  /
**[逐步采用到项目中](GettingStarted/AddtoExistingMonorepo.md)** 

## 特点

Turborepo利用先进的构建系统技术来加快开发速度，**包括在你的本地机器和CI/CD上的开发**。

**[同样的工作不要做两次](CoreConcepts/CachingTasks.md)** / 
**[最大限度的多任务处理](CoreConcepts/Monorepos/RunningTasks.md)** 

## 项目

Turborepo开箱即用，可以与 `npm `、 `pnpm`和 `yarn`等单版本工具一起使用。如果你觉得你的monorepo拖累了你，可能是时候使用Turborepo了。

**[为什么选择 Turborepo?](CoreConcepts/Monorepos.md)** / 
**[阅读项目手册](Handbook/README.md)** 

## 实例

你也可以克隆一个Turborepo启动库，让你的项目有一个初步的发展。更多的例子和启动程序，请看 [GitHub 上的 Turborepo 例子目录](https://github.com/vercel/turbo/tree/main/examples)。

**DEMO1:[Basic](https://github.com/vercel/turbo/tree/main/examples/basic)** / 
**DEMO2:[Design System](https://github.com/vercel/turbo/tree/main/examples/design-system)** / 
**DEMO3:[With Tailwind CSS](https://github.com/vercel/turbo/tree/main/examples/with-tailwind)** / 
**DEMO4:[Kitchen Sink](https://github.com/vercel/turbo/tree/main/examples/kitchen-sink)** /