# 前端项目团队规范统一——以 vue 为例

协作项目中团队的代码规范如何统一是一个问题，幸好前端中有各种各样的工具辅助实现统一规范。

本文一共涉及以下工具 `husky` `lint-staged` `eslint` `prettier` `commitlint` `web-vitals`。本文仅涉及基础配置，高级配置请到以下官方文档查阅

- [Husky - Git hooks (typicode.github.io)](https://typicode.github.io/husky/#/)
- [okonet/lint-staged: 🚫💩 — Run linters on git staged files (github.com)](https://github.com/okonet/lint-staged)
- [Find and fix problems in your JavaScript code - ESLint - Pluggable JavaScript Linter](https://eslint.org/)
- [Prettier · Opinionated Code Formatter](https://prettier.io/)
- [commitlint - Lint commit messages](https://commitlint.js.org/)
- [Web 指标](https://web.dev/vitals/)

配置完成的项目参考：[yuzheng14/vite-base-vue3: This template should help get you started developing with Vue 3 in Vite, husky, lint-staged, eslint, prettier, commitlint and web-vitals. (github.com)](https://github.com/yuzheng14/vite-base-vue3)

如果你只是想快速配置不求过程也可以直接 clone 此 repo

<details>
	<summary><strong>目录</strong></summary>

[TOC]

</details>

## 整体思路

其中 `husky` `lint-staged` `eslint` `prettier` `commitlint` 用于代码质量规范审查，`web-vitals` 用于检测页面性能，用于性能优化

`husky` 用于调用 git hook，将一些指令添加到 git hook中

`lint-staged` 用于对暂存区的文件进行匹配，根据匹配到的规则执行一定的指令。因为每回使用 `eslint` 或 `prettier` 对整个项目进行代码审查非常消耗时间，尤其是大项目，而已经提交的代码肯定已经审查好了，而未暂存的代码不会提交上去，所以只需要对暂存区的文件执行代码审查即可。

`eslint` 是一个对 `js` 代码语法和格式检查的工具

`prettier` 是一个代码格式化工具，因为 `eslint` 的格式化并不完善，所以需要使用 `prettier` 进行补充

`commitlint` 用于对提交的 `git commit message` 进行审查，查看是否符合配置的规范

## 搭建项目

使用 `vite` 快速搭建一个 `vue3` 项目

```shell
# npm 6.x
npm create vite@latest <your-vue-app> --template vue-ts

# npm 7+, extra double-dash is needed:
npm create vite@latest <your-vue-app> -- --template vue-ts

# yarn
yarn create vite <your-vue-app> --template vue-ts

# pnpm
pnpm create vite <your-vue-app> --template vue-ts

```

打开项目并安装依赖

```shell
cd <your-vue-app>
# npm 
npm i

# yarn
yarn

#pnpm
pnpm i
```

## 安装部分依赖

```shell
# npm
npm install --save-dev husky @commitlint/config-conventional @commitlint/cli lint-staged prettier

# yarn
yarn add -D husky @commitlint/config-conventional @commitlint/cli lint-staged prettier

# pnpm 
pnpm add -D husky @commitlint/config-conventional @commitlint/cli lint-staged prettier
```

## `husky`

`husky` 用于将 `lint-staged` 和 `commitlint` 配置到 `git hook` 中

初始化 git （没有 git 哪来的 `git hook` ？）

```shell
git init
```

激活 `husky`

```shell
# npx
npx husky install

# yarn
yarn husky install

# pnpm
pnpm husky install
```

为了使 clone 下来项目后自动激活 `husky`，需要运行以下指令或者直接在 `package.json` 里添加 `prepare` `script`

```shell
npm pkg set scripts.prepare="husky install"
```

```json
{
    ...
    scripts: {
        ...
        "prepare": "husky install"
        ...
    }
    ...
}
```

创建钩子

```shell
# npx
npx husky add .husky/pre-commit "npx lint-staged"

# yarn
yarn husky add .husky/pre-commit "yarn lint-staged"

# pnpm
pnpm husky add .husky/pre-commit "pnpm lint-staged"
```

可以看到项目目录的 `./.husky` 路径下多了个 `pre-commit` 文件

```shell
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged

```

这个就是我们的钩子了

同理创建 `commitlint` 使用的钩子

```shell
# npx
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'

# yarn
yarn husky add .husky/commit-msg 'yarn commitlint --edit ${1}'

# pnpm
pnpm husky add .husky/commit-msg 'pnpm commitlint --edit ${1}'
```

如果需要其他的钩子，可以输入以下指令查看可以有哪些钩子，按照上边的方式添加新钩子

```shell
ls .git/hooks
```

## `commitlint`

创建 `commitlint.config.js` | `.commitlintrc.js` （如果配合 `cz-git` 或者 `czg` 则拓展名需要改为 `.cjs`）

```shell
touch .commitlintrc.cjs
```

写入以下代码

```javascript
// 用于开启代码提示，如果不使用 cz-git 或者 czg 可以创建 ts 文件
// 详情参见官方文档
/** @type {import('cz-git').UserConfig} */
module.exports = {
  // 官方推荐配置
  extends: ['@commitlint/config-conventional'],
  // 自定义配置
  rules: {
    'type-enum': [
      // 级别 0 忽略，1 warning，2 error
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'test', 'chore'],
    ],
  },
}
```

## `lint-staged`

创建 `lint-staged` 的配置文件，`.lintstagedrc.json` | `.lintstagedrc.yaml` | `.lintstagedrc.yml` | `.lintstagedrc.mjs` | `lint-staged.config.mjs` | `.lintstagedrc.cjs` | `lint-staged.config.cjs` | `lint-staged.config.js` or `.lintstagedrc.js` 均可，此处因为需要配置忽略 `eslint` 忽略的文件，所以以 `.lintstagedrc.js` 为例，其他格式请以官方文档为准

```javascript
// 需要先安装 eslint
import { ESLint } from 'eslint'
/**
 * 去除暂存区文件中被 eslint 设置为忽略的文件
 * @param {string[]} files 暂存区的文件名数组
 * @returns 去除忽略的暂存区文件名数组
 */
const removeIgnoredFiles = async (files) => {
  // eslint >= 7 后使用此 API
  const eslint = new ESLint()
  // 获取是否被忽略的 boolean 数组
  const isIgnored = await Promise.all(
    files.map((file) => {
      return eslint.isPathIgnored(file)
    })
  )
  // 根据 isIgnored 数组过滤文件
  const filteredFiles = files.filter((_, i) => !isIgnored[i])
  return filteredFiles.join(' ')
}

// 导出 lint-staged 的配置
export default {
  // 对于 js 或者 ts 文件等使用 eslint 进行审查并直接覆盖文件
  '**/*.{js,jsx,ts,tsx,vue}': async (files) => {
    const filesToLint = await removeIgnoredFiles(files)
    return [`eslint --max-warnings=0 --fix ${filesToLint}`]
  },
  // 对于 样式 json md html yaml 使用 prettier 进行格式化并直接覆盖文件
  '*.{css,scss,json,md,html,yml}': ['prettier --write'],
}
```

## `eslint`

执行以下指令，按照提示进行选择

```shell
npm init @eslint/config
```

### `eslint-config-prettier`

因为 `eslint` 中的代码格式化与 `prettier` 有冲突，所以需要禁用 `eslint` 中与 `prettier` 有冲突的规则，这里可以直接使用 `eslint-conifg-prettier`

```shell
# npm
npm install --save-dev eslint-config-prettier

# yarn
yarn add -D eslint-config-prettier

# pnpm
pnpm add -D eslint-config-prettier
```

在 `eslint` 的配置文件中的 `extend` 中加入 `prettier` 字段，并保证 `prettier` 字段在最后，以确保覆盖所有的规则。

```json
{
    ...
    "extends": [
        ...
        "prettier"
    ],
    ...
}
```

> ℹ️ 注意：网上其他教程可能会说需要在 `extend` 中加入类似 `prettier/vue` 的值来覆盖其他插件，但是从 v8 开始只需要写入 `prettier`，这包括了所有的插件

### `eslint-plugin-prettier`

除此之外，也可以使用 `eslint-plugin-prettier`，这个插件会将 `prettier` 作为 `eslint` 的规则运行，并将报错传递到 `eslint` 进行显示。首先安装该依赖

```shell
# npm
npm install --save-dev eslint-plugin-prettier

# yarn
yarn add -D eslint-plugin-prettier

# pnpm
pnpm add -D eslint-plugin-prettier
```

> ℹ️ 注意：安装 `eslint-plugin-prettier` 不会安装 `eslint` `prettier` 以及 `eslint-config-prettier`，需要手动进行安装

然后在配置文件中加入以下内容

```json
{
    ...
    "plugins": ["prettier"],
    "rules": {
    	"prettier/prettier": "error"
  	...
}
```

这个插件搭配 `eslint-config-prettier` 效果更好，只需将配置文件改成

```json
{
    ...
  	"extends": ["plugin:prettier/recommended"]
    ...
}
```

这个配置等价于以下配置

```json
{
  	"extends": ["prettier"],
  	"plugins": ["prettier"],
  	"rules": {
    	"prettier/prettier": "error",
    	"arrow-body-style": "off",
    	"prefer-arrow-callback": "off"
  	}
}
```

## `prettier`

因为在 `eslint` 中已经配置过 `prettier` 了，所以这里只需要配置好 `prettier` 的规则就可以了。`prettier` 的配置文件可以是 `js` `yaml` `json` `toml` 中的一种，以 `.prettierrc` 为名（如 `.prettierrc.json`）

```json
{
  "trailingComma": "es5",
  "tabWidth": 4,
  "semi": false,
  "singleQuote": true
}
```

这里的配置需要根据团队规范进行配置，具体配置项参考官方文档

## `web-vitals`

此处参考 `create-react-app` 中对 `web-vitals` 的使用。首先在项目顶层新建 `webVitals.js` 文件

```shell
touch webVitals.js
```

将以下代码写入该文件

```javascript
/**
 * 传入参数（函数），将报告的结果传入该参数执行
 * @param {function} onPerfEntry 用于报告性能测试的函数
 */
const reportWebVitals = (onPerfEntry) => {
  // 如果参数存在并且是函数
  if (onPerfEntry && onPerfEntry instanceof Function) {
    // 懒加载并获取到五种性能指标，调用报告性能测试结果的函数
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(onPerfEntry)
      getFID(onPerfEntry)
      getFCP(onPerfEntry)
      getLCP(onPerfEntry)
      getTTFB(onPerfEntry)
    })
  }
}

export default reportWebVitals
```

## 容易掉的坑

`vue` 使用 `typescript` 时在配置 `eslint` 时不要选择 `code-style` 审查

并且需要将 `eslint` 的配置文件如下部分配置，否则无法正确识别 `.vue` 文件格式

```json
{
    ...
    "parser": "vue-eslint-parser",
  	"parserOptions": {
    	"ecmaVersion": "latest",
    	"parser": "@typescript-eslint/parser",
    	"sourceType": "module"
  	},
    ...
}
```

并将 `src/vite-env.d.ts` 文件加入 `.eslintignore` 配置文件中以忽略该文件（目前还不知道该如何改正该文件）
