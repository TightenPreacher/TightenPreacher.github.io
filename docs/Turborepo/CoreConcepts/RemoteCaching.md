# 远程缓存

Turborepo的[任务缓存](https://turbo.build/repo/docs/core-concepts/caching)可以节省大量的时间，不会把同样的工作做两次。

但是有一个问题 - **缓存是在你的机器上的**。当你在使用CI时，这可能会导致大量的重复工作。

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flocal-caching.d097807f.png&w=1920&q=75)

由于Turborepo默认只缓存到本地文件系统，所以即使所有的任务输入都是相同的，同一个任务（`turbo run build`）也必须在**每台机器上重新执行**（由你，由你的队友，由你的CI，由你的PaaS等等）-这就**浪费了时间和资源**。

## 一个单一的、共享的缓存

如果你可以在整个团队（甚至是你的CI）中共享一个Turborepo缓存，会怎么样？

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fremote-caching.5c826549.png&w=1920&q=75)

通过与[Vercel](https://turbo.build/repo/docs/core-concepts/remote-caching#vercel)这样的供应商合作，Turborepo可以安全地与远程缓存--存储任务结果的云服务器进行通信。

This can save enormous amounts of time by **preventing duplicated work across your entire organization**.
这可以通过**防止整个组织的重复工作**而节省大量的时间。

远程缓存是Turborepo的一个强大功能，但强大的功能也伴随着巨大的责任。首先确保你的缓存是正确的，并仔细检查环境变量的处理。也请记住Turborepo将日志作为人工制品处理，所以要注意你在控制台打印的内容。

## Vercel

### 对于本地开发 //For Local Development

如果你想把本地的turborepo链接到远程缓存，首先用你的Vercel账户认证Turborepo CLI:

```sh
npx turbo login
```

接下来，将你的Turborepo与你的远程缓存连接起来:

```sh
npx turbo link
```

一旦启用，对你目前正在缓存的工作区做一些修改，然后用`turbo run`对其运行任务。你的缓存工件现在将被储存在本地 _和_ 你的远程缓存中。

为了验证，请用以下方法删除您的本地Turborepo缓存:

### unix
```sh
  rm -rf ./node_modules/.cache/turbo
```

### win
```sh
  rd /s /q "./node_modules/.cache/turbo"
```

然后再次运行相同的构建。如果工作正常，`Turbo`不应该在本地执行任务，而是从你的远程缓存中下载日志和工件，并将它们回放给你。

### Vercel构建的远程缓存 //Remote Caching on Vercel Builds

如果你在Vercel上构建和托管你的应用程序，一旦你使用`turbo`，远程缓存就会自动为你设置。你需要更新你的[构建设置](https://vercel.com/docs/concepts/deployments/configure-a-build)，以便用`turbo`构建。

请参考[Vercel文档](https://vercel.com/docs/concepts/git/monorepos#turborepo?utm_source=turbo.build&utm_medium=referral&utm_campaign=docs-link)中的说明。

### 工作的完整性和真实性验证 //Artifact Integrity and Authenticity Verification

您可以启用Turborepo，在上传工件到远程缓存之前用秘钥对其进行签名。Turborepo在工件上使用`HMAC-SHA256`签名，使用您提供的密匙。Turborepo会在下载时验证远程缓存工件的完整性和真实性。任何验证失败的工件将被忽略，并被Turborepo视为缓存丢失。

要启用这个功能，在你的 `turbo.json` 配置中设置 `remoteCache` 选项，包括`signature: true`。然后通过声明 `TURBO_REMOTE_CACHE_SIGNATURE_KEY` 环境变量来指定你的密匙。

```jsonc
{
  "$schema": "https://turbo.build/schema.json",
  "remoteCache": {
    // Indicates if signature verification is enabled.
    "signature": true
  }
}
```

## 自定义远程缓存 //Custom Remote Caches

您可以自行托管自己的远程缓存，也可以使用其他远程缓存服务提供商，只要他们符合Turborepo的远程缓存服务器API。

你可以通过指定 `--api` 和 `--token` 标志来设置远程缓存域，其中 `--api` 是主机名， `--token` 是一个承载令牌。

```sh
turbo run build --api="https://my-server.example.com" --token="xxxxxxxxxxxxxxxxx"
```

你可以在这里看到[需要的](https://github.com/vercel/turbo/blob/main/cli/internal/client/client.go)端点/请求。