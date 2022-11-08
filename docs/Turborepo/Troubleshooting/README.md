# 故障排除

本指南旨在帮助您调试Turborepo的构建和配置问题。

## 我的依赖关系没有被正确构建

- 在构建应用程序之前，你是否正确地捆绑和转码了依赖关系?
  - 例如，像 `tsc`, `tsup`, `esbuild`, `babel`, 和 `swc` 这样的库会将较新的JavaScript功能转换回 "纯 "JavaScrip。
  - 如果你使用 Next.js, 你可能使用 `next-transpile-modules`. 确保你在 `next.config.js` 里面添加了依赖关系的名称。([example](https://github.com/vercel/turbo/blob/main/examples/basic/apps/docs/next.config.js#L1))
- 你是否在依赖关系的 `package.json` 中列出了`files`以指向正确的文件?

## 我的类型没有被发现

- 你是否在依赖关系的 `package.json` 里面指定了`types`或`typing` 以指向`.d.ts` 文件?
- 你是否修改或设置了自定义的 `tsconfig.json` `paths`?
  - 它们对你的应用程序有正确的文件夹结构吗?
  - 它们是否为元框架、捆绑器或转译工具正确配置了?

## 我没有看到任何缓存点击

-在构建过程中，是否有任何源代码被生成，而没有被git检查到?
  - 这将改变Turborepo用于存储构建输出的指纹信息.
    - 在你的Turborepo [管道](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks#defining-a-pipeline)中是否[正确指定了缓存输出](https://turbo.build/repo/docs/core-concepts/caching#configuring-cache-outputs)?
  - 管道设置不被继承或合并，所以它们需要在[特定工作区的任务](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks#specific-workspace-tasks)中重新指定（例如，`web#build`**不**继承`build`的管道设置）
    - [相关的内联环境变量是否被计算在内?](https://turbo.build/repo/docs/core-concepts/caching#alter-caching-based-on-environment-variables-and-files)
  - 为了验证，在`turbo run <task>`中加入`-vvv`，在verbose模式下运行`turbo`，看看哪些环境变量被包含在哈希值中。

## 我看到了缓存的点击率，但我的构建被破坏了

- 在你的Turborepo [管道](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks#defining-a-pipeline)中是否[正确指定了缓存输出](https://turbo.build/repo/docs/core-concepts/caching#configuring-cache-outputs)?
  - 管道设置不被继承或合并，所以它们需要在[特定工作区的任务](https://turbo.build/repo/docs/core-concepts/monorepos/running-tasks#specific-workspace-tasks)中重新指定（例如，`web#build`**不**继承`build`的管道设置）

## 我的构建中缓存了错误的环境变量

- [相关的内联环境变量是否被计算在内?](./core-concepts/caching#alter-caching-based-on-environment-variables-and-files)
  - 为了验证, 在`turbo run <task>`中加入`-vvv`，在verbose模式下运行`turbo`，看看哪些环境变量被包含在hash中
