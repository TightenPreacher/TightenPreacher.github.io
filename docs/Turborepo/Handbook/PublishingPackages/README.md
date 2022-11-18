
# 发布软件包 //Publishing Packages

如果有合适的工具，从monorepo发布软件包到`npm`可以是一个非常令人满意和顺利的体验。

如果你想把你的monorepo的一些工作区作为包发布到`npm`，你应该遵循这个设置。如果你不需要发布到`npm`，你应该使用一个[内部包](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages)来代替。它们更容易设置和使用。

## 工具 //Tools

你需要设置一些工具来让它工作:

首先，你需要一个[bundler](https://turbo.build/repo/docs/handbook/publishing-packages/bundling)来把你的代码变成[`CommonJS`](https://en.wikipedia.org/wiki/CommonJS)，这是`npm`上最常用的格式。你还需要设置一个开发脚本，这样你就可以在本地开发的工作空间上工作。

最后，你将需要一个用于[发布和版本管理](https://turbo.build/repo/docs/handbook/publishing-packages/versioning-and-publishing)的工具。这将处理提升你的monorepo的包的版本 _和_ 发布到npm。