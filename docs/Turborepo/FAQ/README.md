# 常见问题

## 使用Turborepo必须要使用远程缓存吗?

不，[远程缓存](https://turbo.build/repo/docs/core-concepts/remote-caching)是可选的。然而，你会发现它非常有用，可以加快团队的开发速度，加快Docker内部的构建速度，还可以节省你机器的空间。

## Turborepo/远程缓存 能否存储我的源代码?

不，Turborepo并不存储源代码。在没有[远程缓存](https://turbo.build/repo/docs/core-concepts/remote-caching)的情况下,任何代码都不会离开你的机器--它只会将文件缓存到本地磁盘
使用Turborepo的远程缓存，您需要负责配置缓存行为，并且只需要设置Turborepo来缓存已编译的文件。请注意Turborepo将所有的日志视为文件，所以这些日志 _将_ 和其他的缓存文件一起存储

## 我必须使用Vercel才能使用Turborepo吗?

Turborepo是一个开源项目，不与任何特定的主机提供商或远程缓存提供商挂钩。默认的远程缓存提供商是Vercel，如果你选择启用它的话。然而，你可以使用你喜欢的任何其他供应商，如果他们支持相同的API。一些开源社区的远程缓存与Turborepo兼容。

## 我可以用Turborepo与Vercel以外的其他远程缓存供应商一起使用吗?

可以的。只要你选择的[远程缓存](https://turbo.build/repo/docs/core-concepts/remote-caching)提供商支持相同的API，你就可以使用Turborepo。

## Turborepo是否收集任何个人身份信息?

由于Turborepo的功能特性，当开源的二进制文件在本地运行时，不会收集任何个人信息。所有缓存的工件都默认存储在您的机器上。此外，`turbo` CLI不会收集任何登录信息或联系信息，所以Turborepo永远不会获得任何个人身份信息。因此，对于任何数据隐私的问题和关注，请参考[Turborepo的隐私政策](https://turborepo.org/privacy)。

## 在使用远程缓存时，Turborepo是否收集任何个人身份信息?

当启用[远程缓存](https://turbo.build/repo/docs/core-concepts/remote-caching)时，默认情况下Turborepo会利用您的Vercel账户在云端缓存工件。因此，对于任何数据隐私的问题和担忧，请参考[Turborepo的隐私政策](https://turborepo.org/privacy)和[Vercel的隐私政策](https://vercel.com/legal/privacy-policy)。如果您使用不同的远程缓存提供商，请参考该提供商的隐私政策。

## 当使用多个Next.js应用程序时，如何在我的Turborepo中保持快速刷新?

[快速刷新](https://nextjs.org/docs/basic-features/fast-refresh) 让你对Next.js应用程序中的React组件所做的编辑有即时的反馈。如果您的 Turborepo 有多个 Next.js 应用程序，您可以使用 `next-transpile-modules` 来确保跨工作区的导入在发生变化时可以使用 Fast Refresh。Turborepo将有效地观察任何编辑和保存时的重建。你可以从这个[例子](https://github.com/vercel/turbo/tree/main/examples/basic)开始，它被设置为处理快速刷新。