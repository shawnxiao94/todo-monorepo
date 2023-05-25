# todo-monorepo-

Monorepo demo

#### 构建 Monorepo 项目

在使用 monorepo 管理项目时，pnpm 是一个优秀的选择，它有更快的安装速度和更少的磁盘空间占用，能够更好地处理多个项目之间的依赖关系。下面是使用 pnpm 搭建 Monorepo 项目的步骤：

全局安装 pnpm：

```

npm install pnpm -g

```

创建 monorepo 仓库

```
mkdir monorepo && cd monorepo
pnpm init

```

在根目录下添加 pnpm-workspace.yaml 文件，告诉 pnpm 哪些目录是工作区，并在安装依赖时将它们链接到一起。

```
packages:
  - 'packages/**'

```

添加项目：

```
mkdir packages && cd packages

# pkg-a
mkdir pkg-a && cd pkg-a
pnpm init

# pkg-b
mkdir ../pkg-b && cd ../pkg-b
pnpm init

```

安装依赖：

```

# 全局安装 -w: --workspace-root
pnpm add typescript -D -w

# 局部安装
pnpm add typescript -D --filter pkg-a

# 互相安装
pnpm add pkg-a -D --workspace --filter pkg-b

```

写点代码：

```
// packages/pkg-a/index.ts
export function sum(n: number, m: number) {
  return n + m
}


// packages/pkg-b/index.ts
import { sum } from 'pkg-a'

sum(1, 2)

```

在 packages/pkg-a/package.json 和 packages/pkg-b/package.json 中添加以下脚本：

```
{
  "scripts": {
    "build": "tsc index.ts"
  }
}

```

在根目录的 package.json 中添加以下脚本：

```
{
  "scripts": {
    "build": "pnpm -r --filter=./packages/* run build"
  }
}

```

根目录运行以下命令：

```

pnpm run build

```

#### 发布 npm 包

在执行以下步骤之前，请确保已经将代码推送到链接的 git 远程仓库中。

使用 bumpp 进行版本控制，创建 tag 标签并推送到 远程 git 仓库

安装 bumpp：

```
pnpm add bumpp -D

```

> ERR_PNPM_ADDING_TO_ROOT  Running this command will add the dependency to the workspace root, which might not be what you want - if you really meant it, make it explicit by running this command again with the -w flag (or --workspace-root). If you don't want to see this warning anymore, you may set the ignore-workspace-root-check setting to true.

```
pnpm config set ignore-workspace-root-check true
```

在根目录的 package.json 中添加以下脚本：

```
{
  "scripts": {
    "release": "bumpp package.json packages/**/package.json"
  }
}

```

在根目录运行以下命令

```
pnpm run release

```

这个命令将自动升级你的 npm 包版本，创建 git 标签，并将更改推送到你的远程 git 存储库。

#### 使用 GitHub Actions 自动发布 npm 包

1.首先，确保 pnpm run release 运行成功，并将更改推送到远程 git 存储库。

2.创建一个名为 .github/workflows/release.yml 的文件。

3.在 release.yml 文件中添加以下代码：

```
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup node
        uses: actions/setup-node@v2
        with:
          node-version: '16'
          registry-url: https://registry.npmjs.org/

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: latest

      - name: Install dependencies
        run: pnpm i

      - name: Build
        run: pnpm run build

      - name: Publish
        run: pnpm publish -r --access public --no-git-checks
        env:
          NODE_AUTH_TOKEN: ${{secrets.NPM_TOKEN}}

      - run: npx changelogithub
        env:
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}

```

4.接下来，在 npm 上获取你的认证令牌。请注意，你需要拥有发布包的权限才能获得令牌。将令牌添加到 GitHub 仓库的 secrets 中。

5.转到你的 GitHub 仓库的 “Settings” 页面，在左侧菜单中，单击 “Secrets and variables / Actions”。单击 “New repository secret” 按钮创建一个新的密钥，名称为 “NPM_TOKEN”，值为你在步骤 4 中获得的认证令牌。

6.现在，当你推送一个带有标签的 commit 到 GitHub 仓库时，GitHub Actions 将会自动运行你的 release.yml 工作流程，并将你的 npm 包发布到 npm 官方网站。

#### 使用 changesets 管理包版本和发布

安装和初始化 changesets

```
pnpm i -Dw @changesets/cli

```

安装完成以后，你可以在项目根目录执行以下命令以快速初始化 changesets

```
pnpm changeset init

```

这时候，你会发现，项目根目录下多了一个 .changeset 目录，其中 config.json 是 changesets 的配置文件。请注意，我们需要把这个目录一起提交到 git 上。

- 发布第一个版本

在模板中，pkg-a、pkg-b 2 个包的版本号都是 1.0.1，我们可以在项目根目录下直接运行 pnpm changeset publish 为三个包发布第一个版本。发布完成后，我们完成了 monorepo 项目的初始化，我们可以把这个改动合并到你的主分支上并提交到远程仓库中

- 生成 changeset 文件

假设现在要进行一个迭代，我们从主分支上切一个 release/0.1.0 分支出来。我们在 packages/a/index.ts 文件中随便添加一行代码，并提交到远程仓库。

pkg-a 代码发生变更了，我们需要发一个版本给用户使用。这时候我们在项目根目录下执行以下命令来选择要发布的包以及包的版本类型（patch、minor、minor，严格遵循 semver 规范）：

```
pnpm changeset

```

changeset 通过 git diff 和构建依赖图来获得要发布的包。我们选择发布 pkg-a：

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a0ef573b78b49648a48b62128f2fd53~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

我们选择更新到 minor 版本：

![img2](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f888adbe1a343eeb5f74e5b9bc6f4d4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

填写 changelog：
![img3](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/747c206e861c4d4f9d08409ddff5449c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

这时候，会发现多出来一个文件名随机的 changeset 文件：

![img4](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/240ad794bf0f4d46b43845eb03783cb5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

这个文件的本质是对包的版本和 Changelog 做一个预存储，我们也可以在这些文件中修改信息。随着不同开发者进行开发迭代积累，changeset 可能会有多个的，比如 pnpm 仓库：

![img5](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02c98bdf74914ae39720a2543cfd7cff~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

这些 changeset 文件是需要一并提交到远程仓库中的。在后面的包发布后，这些 changeset 文件是会被自动消耗掉的。

发布测试版本

假设现在我们要发布一个测试的版本来看下功能是否正常 work，我们可以使用 changeset 的 Prereleases 功能。

通过执行 pnpm changeset pre enter <tag> 命令进入先进入 pre 模式。

```
pnpm changeset pre enter alpha   # 发布 alpha 版本
pnpm changeset pre enter beta    # 发布 beta 版本
pnpm changeset pre enter rc      # 发布 rc 版本

```

这里我运行第二条命令，选择发布 beta 版本。

然后执行 pnpm changeset version 修改包的版本：

![img6](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9bf5dbe6623425196a439e0f6707a20~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

![img7](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df042debc8d5419880bdd715ca504d58~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

可以看到 pkg-a 的版本改成了 0.1.0-beta.0，pkg-b 依赖的 pkg-a 版本也对应修改了。

这时执行 pnpm run build && pnpm changeset publish 发布 beta 版本：

![img8](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb65909b0eda4335b64942b503f73ace~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

完成版本发布之后，退出 Prereleases 模式：

```
pnpm changeset pre exit

```

这时，我们需要把变更的内容提交到远程仓库中，一方面，便于后面查看每次测试版本发布的变更记录；另一方面，changesets 默认不会到 npm 中查找当前包最新的测试包版本号并自动加 1，它是根据当前仓库的测试包版本号再往上递增生成新的版本号。

发布正式版本

测试版本验证完成以后，执行以下命令把包版本修改成正式版本：

```
pnpm changeset version

```

![img9](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c4e7d32f3c1412598e3a2f5be3c7489~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

然后我们执行以下命令发布正式版本：

```
pnpm changeset publish

```

changeset 会检查当前工作区中所有包的版本是否已经被发布过，如果没有则自动发布。

![img10](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3130b7aab260482a81693374efb2a1ee~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

结合 GitHub Actions 实现自动化版本修改和发布

我们完成一次版本迭代，可能会包含多个功能和多个 PR，可能有多个开发者参与开发。一般会基于主分支切一个 release 分支作为开发分支。然后每个开发者基于 release 分支切一个新分支完成他的功能，Code Review 通过以后，先合并到 release 分支。整体测试完成以后，release 分支再合并到主分支，永远保证主分支的代码是足够稳定的。

下图是 alibaba/ice 仓库的 Git Graph:

![img11](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17cf9494fb594dc7b4770e2ae96900e9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

自动修改版本

结合 GitHub Actions 添加 version.yml 文件到 .github/workflows 目录，当有代码合并到 release 分支时，将由 changeset 自动提交 PR 把子包的版本更新到正式版本。

```
name: Version

on:
  push:
    branches:
      - release-next

jobs:
  version:
    name: Version
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16]

    steps:
      - name: Checkout Branch
        uses: actions/checkout@v3

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 7

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'pnpm'

      - name: Install Dependencies
        run: pnpm install

      - name: Create Release Pull Request
        uses: changesets/action@v1
        with:
          version: pnpm changeset version
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

1[img12](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff68e0d638f54572ba202765199e6814~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

当 release 分支所有要迭代的功能完成以后，我们可以把上面提交的版本和 Changelog 修改的 PR（分支是 changeset-release/release）合并到 release 分支上。

自动发布

当 release 分支上的代码测试通过后，将合并到 main 分支上。可结合 GitHub Actions 和 changesets publish 在 ci 流程上发布包到最新版本。

```
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16]

    steps:
      - name: Checkout Branch
        uses: actions/checkout@v3

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 7

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'pnpm'

      - name: Install Dependencies
        run: pnpm install

      - name: Build Packages
        run: pnpm run build

      - name: Publish to npm
        id: changesets
        uses: changesets/action@v1
        with:
          publish: pnpm changeset publish
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}


```

本文先试介绍了 monorepo 方案要解决的问题并分别介绍了 pnpm 和 changesets 工具是怎么解决的。然后基于 pnpm 和 changesets 两个工具搭建 monorepo 项目，并结合 GitHub Actions 实现自动化版本修改和发布。
