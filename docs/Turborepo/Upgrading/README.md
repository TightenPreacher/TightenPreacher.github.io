# 升级到 to Turborepo v1.x

Turborepo已被Vercel收购!随着这一消息的公布, **Vercel正在开源`turbo` CLI，并在过渡期内为所有账户免费提供远程缓存**

现有的Turborepo客户应尽快将他们的`turbo` CLI升级到v1.x，并迁移到Vercel（说明如下）。1.x之前的`turbo` CLI的早期版本将不再被维护。beta.turborepo.com上的新账户创建已被禁用。beta.turborepo.com的仪表板和远程缓存服务将在2022年1月15日关闭，旧版本将无法安装。

**所有现有的远程缓存工件也将在这个时候被删除**。

下面是Turborepo现有用户的迁移步骤指南。如果您遇到困难，请在社区 [Discord](https://turbo.build/discord) 中联系我们，或者在 [GitHub](https://github.com/vercel/turbo) 上提交问题。再次感谢您一直以来的支持，我们一起开始了Turborepo的精彩新篇章。

---

## 1. 清理工作 //Cleaning up

为了保持良好的卫生，确保你注销`Turbo`以删除旧的证书:

```sh
yarn turbo logout
```

如果它存在，也从你的monorepo根部删除`.turbo`目录

```sh
rm -rf .turbo
```

## 2. 安装最新版本的`Turbo` //Install the latest release of `turbo` 

安装最新版本的`turbo`:

```sh
yarn add turbo --save-dev --ignore-workspace-root-check
```

## 3. 设置远程缓存 //Setup Remote Caching

如前所述，Turborepo现在通过[Vercel](https://vercel.com?utm_source=turbo.build&utm_medium=referral&utm_campaign=docs-link)提供零配置的远程缓存。在这个过渡时期，所有Vercel计划的远程缓存都是免费的。每个Vercel账户都有一个共享的远程缓存。这个缓存在所有环境（开发、预览和生产）中共享。

**重要提示**：turborepo.com允许每个团队有多个缓存（即项目）（通过`--project`标志表示）。通过Vercel上的V1.x缓存，每个Vercel账户（用户或团队）都有一个共享的远程缓存。如果你的团队积极使用多个turborepo.com项目，请在[Discord](https://turbo.build/discord)告诉我们。

请注意，我们不会将缓存工件迁移到Vercel。在您迁移期间，当您在Vercel或自定义缓存infra上为您的远程缓存重新注水时，我们对构建速度较慢表示歉意。

## 4. 本地开发 //Local Development

如果你是使用远程缓存进行本地开发，升级将需要一两分钟。要开始，请登录Vercel CLI:

```sh
npx turbo login
```

现在我们可以通过运行 Vercel 设置远程缓存:


```sh
npx turbo link
```

按照提示，选择希望连接的Vercel账户（用户或团队）.

### 在Vercel //On Vercel

- 如果你已经同时使用了Turborepo和Vercel，请从所有项目中删除`TURBO_TOKEN`、`TURBO_TEAM`和`TURBO_PROJECT`环境变量。这些变量现在由Vercel自动替你设置

- 在你的Vercel项目设置和/或`package.json`脚本中删除使用`--team`、`--token`和`--project` CLI标志。

### 在其他CI/CD //On other CI/CD

- 用一个新的[Vercel个人访问令牌](https://vercel.com/account/tokens)替换你的turborepo.com个人访问令牌，并更新`TURBO_TOKEN`环境变量或使用`-token` CLI标志的同等用法。
- 移除`TURBO_PROJECT`环境变量，并移除所有对`--project` CLI标志的使用。这已经被废弃了
- 更新`TURBO_TEAM`环境变量和`--team` CLI标志的值为你的Vercel账户的slug（即`https://vercel.com/<slug>`）

### 获得帮助 //Getting Help

如果你在升级时遇到困难，请在[GitHub](https://github.com/vercel/turbo)上提交一个问题。如果你在Vercel的远程缓存中遇到困难，请在[Discord](https://turbo.build/discord)中联系。