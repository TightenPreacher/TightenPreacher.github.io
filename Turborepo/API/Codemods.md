# Turborepo Codemods

Turborepo提供了Codemod转换和自动迁移脚本，当某个功能被废弃时，可以帮助你升级Turborepo代码库。

Codemods是以编程方式在你的代码库上运行的转换。这使得大量的变化可以被应用，而不需要手动浏览每个文件。

## 使用方法 

```sh
npx @turbo/codemod <transform> <path>
```

- `transform` - 变换的名称，见下面的可用变换.
- `path` - 进行转换的文件或目录
- `--dry` - 做一次模拟运行，不编辑任何代码
- `--print` - 打印改变后的输出，以供比较

## Turborepo 1.x

### `add-package-manager`

转换根`package.json`，使`packageManager`关键字成为检测到的软件包管理器（`yarn`、`npm`、`pnpm`）和版本（例如`yarn@1.22.17`）。现在[Node.js支持](https://nodejs.org/dist/latest-v17.x/docs/api/packages.html#packagemanager)这个键，Turborepo使用这个键来快速检测软件包管理器（而不是仅仅从文件系统中推断）。

例如, 来自 Yarn v1:

```json
// Before
{
  "name": "turborepo-basic",
  "version": "0.0.0",
  "private": true,
  "workspaces": ["apps/*", "packages/*"]
  // ...
}
```

```diff
{
  "name": "turborepo-basic",
  "version": "0.0.0",
  "private": true,
+  "packageManager": "yarn@1.22.17",
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

#### 使用方法

转到你的项目:

```sh
cd path-to-your-turborepo/
```

运行 codemod:

```sh
npx @turbo/codemod add-package-manager
```

### `create-turbo-config`

根据`package.json`中的 `"turbo"`键，在项目的根部创建`turbo.json`文件。随后，`"turbo"`键将从`package.json`中删除。

比如说:

```json
// Before, package.json
{
  "name": "Monorepo root",
  "private": true,
  "turbo": {
    "pipeline": {
      ...
    }
  },
  ...
}
```

```diff
// After, package.json
{
  "name": "Monorepo root",
  "private": true,
-  "turbo": {
-    "pipeline": {
-      ...
-    }
-  },
  ...
}

// After, turbo.json
+{
+  "$schema": "https://turborepo.org/schema.json",
+  "pipeline": {
+    ...
+  }
+}
```

#### 使用说明

转到你的项目:

```sh
cd path-to-your-turborepo/
```

运行 codemod:

```sh
npx @turbo/codemod create-turbo-config
```

### `migrate-env-var-dependencies`

Migrates all environment variable dependencies in `turbo.json` from `dependsOn` and `globalDependencies` to `env` and `globalEnv` respectively.

将`turbo.json`中的所有环境变量依赖从 `dependsOn` 和 `globalDependencies` 分别迁移到 `env` 和 `globalEnv`。

比如说:

```json
// Before, turbo.json
{
  "$schema": "https://turborepo.org/schema.json",
  "globalDependencies": [".env", "$CI_ENV"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build", "$API_BASE"],
      "outputs": [".next/**"]
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false
    }
  }
}
```

````diff
// After, turbo.json
{
  "$schema": "https://turborepo.org/schema.json",
- "globalDependencies": [".env", "$CI_ENV"],
+ "globalDependencies": [".env"],
+ "globalEnv": ["CI_ENV"],
  "pipeline": {
    "build": {
-     "dependsOn": ["^build", "$API_BASE"],
+     "dependsOn": ["^build"],
+     "env": ["API_BASE"],
      "outputs": [".next/**"],
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false
    }
  }
}

#### 使用说明

转到你的项目:

```sh
cd path-to-your-turborepo/
````

运行 codemod:

```sh
npx @turbo/codemod migrate-env-var-dependencies
```
