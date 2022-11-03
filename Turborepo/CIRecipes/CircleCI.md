# 在CircleCI中使用Turborepo

下面的例子展示了如何在[CircleCI](https://circleci.com/)中使用Turborepo。

对于一个给定的根目录`package.json`:

```json
{
  "name": "my-turborepo",
  "scripts": {
    "build": "turbo run build",
    "test": "turbo run test"
  },
  "devDependencies": {
    "turbo": "1.2.5"
  }
}
```

和 `turbo.json`:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": []
    },
  }
}
```

在你的存储库中创建名为`.circleci/config.yml`的文件，内容如下

### npm
```yaml
    version: 2.1
    orbs:
      node: circleci/node@5.0.2
    workflows:
      test:
        jobs:
          - test
    jobs:
      test:
        docker:
          - image: cimg/node:lts
        steps:
          - checkout
          - node/install-packages
          - run:
            command: npm run build
          - run:
            command: npm run test
```

### yarn
```yaml
    version: 2.1
    orbs:
      node: circleci/node@5.0.2
    workflows:
      test:
        jobs:
          - test
    jobs:
      test:
        docker:
          - image: cimg/node:lts
        steps:
          - checkout
          - node/install-packages:
            pkg-manager: yarn
          - run:
            command: yarn build
          - run:
            command: yarn test
```

### pnpm
```yaml
    version: 2.1
    orbs:
      node: circleci/node@5.0.2
    workflows:
      test:
        jobs:
          - test
    jobs:
      test:
        docker:
          - image: cimg/node:lts
        steps:
          - checkout
          - node/install-packages:
          - run:
            command: npm i -g pnpm
          - run:
            command: pnpm build
          - run:
            command: pnpm test
```

## 远程缓存 //Remote Caching

to make them available to your `turbo` commands.
为了使用CircleCI的远程缓存，请在你的CircleCI项目中添加以下环境变量，以使它们对你的`Turbo`命令可用。

- `TURBO_TOKEN` - 用于访问远程缓存的承载令牌
- `TURBO_TEAM` - 项目所属的账户

为了使用Vercel远程缓存，你可以通过几个步骤获得这些变量的值

1. 在[Vercel仪表盘](https://vercel.com/account/tokens)上为你的账户创建一个范围内的访问令牌

![Vercel Access Tokens](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-tokens.2a1aed6c.png&w=3840&q=75)
![Vercel Access Tokens](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-create-token.0d4b01c1.png&w=3840&q=75)

将该值复制到一个安全的地方。你一会儿就会需要它。

2. 进入你的CircleCI仓库设置，然后点击**Environment Variables**标签。创建一个名为`TURBO_TOKEN`的新变量，并输入范围访问令牌的值

![CircleCI Environment Variables](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fcircleci-environment-variables.77521c79.png&w=1920&q=75)
![CircleCI Create Environment Variables](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fcircleci-create-environment-variables.90ea0fd3.png&w=1920&q=75)

3. 制作第二个秘钥，称为`TURBO_TEAM`，并输入你的团队的Vercel URL的值， _不含_ `vercel.com/`。你的团队URL可以在仪表盘上的团队一般项目设置中找到。

   如果你使用的是爱好计划，你可以使用你的用户名。你的用户名可以在你的[Vercel个人账户设置](https://vercel.com/account)中找到

![Vercel Account Slug](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-slug.20565060.png&w=1200&q=75)

4. CircleCI自动将存储在项目设置中的环境变量加载到CI环境中。对CI文件不需要修改。