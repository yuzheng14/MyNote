# vue 项目中菜单如何与路由结合

中后台项目中与路由结合的侧边菜单非常常见，但如何能够优雅的自动生成菜单并进行路由导航却还是需要耗费一定精力的。本文章将以组件的形式，探讨菜单（不仅仅是中后台项目）应如何与路由相结合。

> 以下的所有指令均在项目根目录执行
>
> 需要更改目录执行指令的操作均通过 `(cd <路径> && <要执行的指令>)` 创建子 shell 的形式执行

## 技术栈

* vue@3
* ant-design-vue@4

## 创建 monorepo 项目

因为此次有两个项目：**菜单组件**和**引用的前端**。所以选用 `monorepo` 的形式。

首先创建 `pnpm` `monorepo` 工作区

```bash
pnpm init
mkdir packages
mkdir packages/components
(cd packages/components && pnpm init)
mkdir packages/example
(cd packages/example && pnpm init)
touch pnpm-workspace.yaml
```

在 `pnpm-workspace.yaml` 文件中写入以下内容

```yaml
packages:
  - packages/*
```

在根目录执行一下 `pnpm i` ，至此项目基础框架搭建完成，项目结构如下

```
.
├── package.json
├── packages
│   ├── components
│   │   └── package.json
│   └── example
│       └── package.json
├── pnpm-lock.yaml
└── pnpm-workspace.yaml
```

## 组件打包框架

> 本小节文件均在 `packages/components` 目录下

本次组件打包采用 `rollup`

首先安装依赖

```bash
pnpm --filter components add vue ant-design-vue
pnpm --filter components add -D rollup @rollup/plugin-typescript @rollup/plugin-node-resolve unplugin-vue rollup-plugin-esbuild typescript tslib vite
```

创建 `tsconfig.json` 文件并写入以下内容

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ES2022",
    "moduleResolution": "Bundler",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}

```

创建 `rollup.config.ts` 文件并写入以下内容

```typescript
import { defineConfig } from 'rollup'
import vue from 'unplugin-vue/rollup'
import { nodeResolve } from '@rollup/plugin-node-resolve'
import esbuild from 'rollup-plugin-esbuild'
import { createRequire } from 'node:module'

const pkg = createRequire(import.meta.url)('./package.json')

export default defineConfig({
  input: './src/index.ts',
  output: {
    file: './dist/index.js',
    format: 'esm',
  },
  plugins: [
    vue(),
    nodeResolve(),
    esbuild({
      loaders: {
        '.vue': 'ts',
      },
    }),
  ],
  external: [pkg.dependencies, pkg.devDependencies, pkg.peerDependencies]
    .filter((v) => v)
    .flatMap(Object.keys),
})

```

创建 `src/global.d.ts` 文件并写入以下内容

```typescript
declare module '*.vue' {
  const components: DefineComponent
  export default components
}
```

创建 `src/MyMenu.vue` 写入以下内容

```vue
<script setup lang="ts">
defineOptions({
  name: 'MyMenu',
})
</script>
```

创建 `src/index.ts` 文件，并写入以下内容

```typescript
export { default as MyMenu } from './MyMenu.vue'
```

在 `package.json` 中写入 `build` 指令

```json
{
  "scripts": {
    "build": "rollup -c --configPlugin typescript"
  },
}
```

现在运行 `pnpm --filter components build` 就可以看到打包结果了

## 前端项目

> 本小节文件路径都在 `packages/example` 下

执行下面的指令创建 `vite` 项目

```bash
(cd packages/example && pnpm create vite . -t vue-ts)
```

执行下面的指令安装工作区依赖

```bash
pnpm --filter example add components
```

删除其他组件后