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
