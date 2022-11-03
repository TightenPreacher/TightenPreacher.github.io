
# 在Travis CI中使用Turborepo //Using Turborepo with Travis CI

下面的例子展示了如何在[Travis CI](https://www.travis-ci.com/)中使用Turborepo。

对于一个给定的根目录`package.json`

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

和一个 `turbo.json`:

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

在你的存储库中创建一个名为`.travis.yml`的文件，内容如下。

### npm

```yaml
    language: node_js
    node_js:
      - lts/*
    install:
      - npm install
    script:
      - npm run build
    script:
      - npm run test
```

### yarn

Travis CI通过`yarn.lock`的存在来检测Yarn的使用。它将自动确保它被安装。

```yaml
    language: node_js
    node_js:
        - lts/*
    install:
        - yarn
    script:
        - yarn build
    script:
        - yarn test
```

### pnpm

```yaml
    language: node_js
    node_js:
      - lts/*
    cache:
      npm: false
      directories:
        - "~/.pnpm-store"
    before_install:
      - curl -f https://get.pnpm.io/v6.16.js | node - add --global pnpm@6.32.2
      - pnpm config set store-dir ~/.pnpm-store
    install:
      - pnpm install
    script:
      - pnpm build
    script:
      - pnpm test
```

欲了解更多信息，请访问关于Travis CI集成的pnpm文档部分，在[此](https://pnpm.io/continuous-integration#travis)查看

## Remote Caching

要在Travis CI中使用远程缓存，请在你的Travis CI项目中添加以下环境变量。

- `TURBO_TOKEN` - 用于访问远程缓存的承载令牌。
- `TURBO_TEAM` - 项目所属的账户

要使用Vercel远程缓存，你可以通过几个步骤获得这些变量的值

1. 在[Vercel仪表盘](https://vercel.com/account/tokens)上为你的账户创建一个范围内的访问令牌


![Vercel Access Tokens](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-tokens.2a1aed6c.png&w=3840&q=75)
![Vercel Access Tokens](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-create-token.0d4b01c1.png&w=3840&q=75)

将该值复制到一个安全的地方。你一会儿就会需要它。


2. 进入你的Travis资源库设置，向下滚动到 _环境变量(Environment Variables)_ 部分。创建一个名为`TURBO_TOKEN`的新变量，并输入你的范围访问令牌的值。

![Travis CI Variables](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Ftravis-ci-environment-variables.537da6d2.png&w=2048&q=75)


3. 制作第二个秘钥，称为`TURBO_TEAM`，并输入你的团队的Vercel URL的值，_不含_ `vercel.com/`。你的团队URL可以在仪表板上的团队一般项目设置中找到

   如果你使用的是爱好计划，你可以使用你的用户名。你的用户名可以在你的[Vercel个人账户设置](https://vercel.com/account)中找到

![Vercel Account Slug](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-slug.20565060.png&w=1200&q=75)



4. Travis CI会自动将存储在项目设置中的环境变量加载到CI环境中。对CI文件不需要修改。
