
# 在GitHub Actions中使用Turborepo

下面的例子展示了如何在[GitHub Actions](https://github.com/features/actions)中使用Turborepo。

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
    }
  }
}
```

在你的存储库中创建名为`.github/workflows/ci.yml`的文件，内容如下

### npm
```yaml
  name: CI

  on:
    push:
      branches: ["main"]
    pull_request:
      types: [opened, synchronize]

  jobs:
    build:
        name: Build and Test
        timeout-minutes: 15
        runs-on: ubuntu-latest
        # To use Remote Caching, uncomment the next lines and follow the steps below.
        # env:
  #  TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
  #  TURBO_TEAM: ${{ secrets.TURBO_TEAM }}
  #  TURBO_REMOTE_ONLY: true

  steps:
    - name: Check out code
      uses: actions/checkout@v3
      with:
        fetch-depth: 2

    - name: Setup Node.js environment
      uses: actions/setup-node@v3
      with:
        node-version: 16
        cache: 'npm'

    - name: Install dependencies
      run: npm install

    - name: Build
      run: npm run build

    - name: Test
      run: npm run test
```

### yarn
```yaml
  name: CI

  on:
    push:
      branches: ["main"]
    pull_request:
      types: [opened, synchronize]

  jobs:
    build:
      name: Build and Test
      timeout-minutes: 15
      runs-on: ubuntu-latest
      # To use Remote Caching, uncomment the next lines and follow the steps below.
      # env:
      #  TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
      #  TURBO_TEAM: ${{ secrets.TURBO_TEAM }}

      steps:
        - name: Check out code
          uses: actions/checkout@v3
          with:
            fetch-depth: 2

        - name: Setup Node.js environment
          uses: actions/setup-node@v3
          with:
            node-version: 16
            cache: 'yarn'

        - name: Install dependencies
          run: yarn

        - name: Build
          run: yarn build

        - name: Test
          run: yarn test
```

### pnpm
```yaml
    name: CI

    on:
      push:
        branches: ["main"]
      pull_request:
        types: [opened, synchronize]

    jobs:
      build:
        name: Build and Test
        timeout-minutes: 15
        runs-on: ubuntu-latest
        # To use Remote Caching, uncomment the next lines and follow the steps below.
        # env:
        #  TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
        #  TURBO_TEAM: ${{ secrets.TURBO_TEAM }}

        steps:
          - name: Check out code
            uses: actions/checkout@v3
            with:
              fetch-depth: 2

          - uses: pnpm/action-setup@v2.0.1
            with:
              version: 6.32.2

          - name: Setup Node.js environment
            uses: actions/setup-node@v3
            with:
              node-version: 16
              cache: 'pnpm'

          - name: Install dependencies
            run: pnpm install

          - name: Build
            run: pnpm build

          - name: Test
            run: pnpm test
```

## 远程缓存 //Remote Caching

要在GitHub Actions中使用远程缓存，需要在GitHub Actions工作流程中添加以下环境变量，使其对你的`turbo`命令可用。

- `TURBO_TOKEN` - 用于访问远程缓存的承载令牌。
- `TURBO_TEAM` - 项目所属的账户

为了使用Vercel远程缓存，你可以通过几个步骤获得这些变量的值

1. 在[Vercel仪表盘](https://vercel.com/account/tokens)上为你的账户创建一个范围内的访问令牌

![Vercel Access Tokens](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-tokens.2a1aed6c.png&w=3840&q=75)
![Vercel Access Tokens](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-create-token.0d4b01c1.png&w=3840&q=75)

将该值复制到一个安全的地方。你一会儿就会需要它。

2. 进入你的GitHub仓库设置，点击**Secrets**，然后点击**Actions**标签。创建一个名为`TURBO_TOKEN`的新变量，并输入范围访问令牌的值

![GitHub Secrets](/images/docs/github-actions-secrets.png)
![GitHub Secrets Create](/images/docs/github-actions-create-secret.png)

3. 制作第二个秘钥，称为`TURBO_TEAM`，并输入你的团队的Vercel URL的值， _不含_ `vercel.com/`。你的团队URL可以在仪表盘上的团队一般项目设置中找到。

   如果你使用的是爱好计划，你可以使用你的用户名。你的用户名可以在你的[Vercel个人账户设置](https://vercel.com/account)中找到

![Vercel Account Slug](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fvercel-slug.20565060.png&w=1200&q=75)

4. 在GitHub Actions工作流程的顶部，为使用`turbo`的作业提供以下环境变量:

```yaml
# ...

jobs:
  build:
    name: Build and Test
    timeout-minutes: 15
    runs-on: ubuntu-latest
    # To use Turborepo Remote Caching, set the following environment variables for the job.
    env:
      TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
      TURBO_TEAM: ${{ secrets.TURBO_TEAM }}

    steps:
      - name: Check out code
        uses: actions/checkout@v2
        with:
          fetch-depth: 2
    # ...
```