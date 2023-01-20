# `yarn` 换源后不生效

[TOC]

## 问题重现

今天克隆下来 GitHub 上的一个项目安装依赖时前面的非常快（因为是从 cached 里有），最后一个却怎么也 fetch 不下来，卡在了 fetching package

## 问题尝试

通过 `--verbose` 参数发现包是从 https://registry.yarnpkg.com/ 下载的，但是配置的源是 https://registry.npmmirror.com/，清理 `cache` 

```shell
yarn cache clean
```

清理完成后重新 `yarn` 发现所有的包都是从官方源下载，速度极慢

## 问题解决

通过 `vscode` 的搜索功能发现，`yarn.lock` 文件内定义依赖的 `resolved` 全部是官方源，于是使用 <kbd>Ctrl</kbd> + <kbd>F</kbd> 将所有的官方源全部替换为国内镜像源，问题解决

