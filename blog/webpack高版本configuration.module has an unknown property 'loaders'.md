

webpack更换高版本后报错

```
[webpack-cli] Invalid configuration object. Webpack has been initialized using a configuration object that does not match the API schema.
 - configuration.module has an unknown property 'loaders'. These properties are valid:
   object { defaultRules?, exprContextCritical?, exprContextRecursive?, exprContextRegExp?, exprContextRequest?, generator?, noParse?, parser?, rules?, strictExportPresence?, strictThisContextOnImports?, unknownContextCritical?, unknownContextRecursive?, unknownContextRegExp?, unknownContextRequest?, unsafeCache?, wrappedContextCritical?, wrappedContextRecursive?, wrappedContextRegExp? }
   -> Options affecting the normal modules (`NormalModuleFactory`).
   Did you mean module.rules or module.rules.*.use?
```

其中关键信息为

```
configuration.module has an unknown property 'loaders'. These properties are valid
```

根据 [- configuration.module has an unknown property 'loader' 问题解决 - 九许尘歌 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zuojiayi/p/9342452.html) 发现是`module.loader`在新版本中改为`modul.ruls`

改正后又报错误

```
[webpack-cli] Invalid configuration object. Webpack has been initialized using a configuration object that does not match the API schema.
 - configuration.module.rules[5] should be one of these:
   ["..." | object { assert?, compiler?, dependency?, descriptionData?, enforce?, exclude?, generator?, include?, issuer?, issuerLayer?, layer?, loader?, mimetype?, oneOf?, options?, parser?, realResource?, resolve?, resource?, resourceFragment?, resourceQuery?, rules?, scheme?, sideEffects?, test?, type?, use? }, ...]
   -> A rule.
   Details:
    * configuration.module.rules[2] has an unknown property 'loaders'. These properties are valid:
      object { assert?, compiler?, dependency?, descriptionData?, enforce?, exclude?, generator?, include?, issuer?, issuerLayer?, layer?, loader?, mimetype?, oneOf?, options?, parser?, realResource?, resolve?, resource?, resourceFragment?, resourceQuery?, rules?, scheme?, sideEffects?, test?, type?, use? }
      -> A rule description with conditions and effects for modules.
    * configuration.module.rules[5] has an unknown property 'loaders'. These properties are valid:
      object { assert?, compiler?, dependency?, descriptionData?, enforce?, exclude?, generator?, include?, issuer?, issuerLayer?, layer?, loader?, mimetype?, oneOf?, options?, parser?, realResource?, resolve?, resource?, resourceFragment?, resourceQuery?, rules?, scheme?, sideEffects?, test?, type?, use? }
      -> A rule description with conditions and effects for modules.
```

其中关键信息为

```
Details:
    * configuration.module.rules[2] has an unknown property 'loaders'. These properties are valid:
```

查阅 webpack 官方文档 [模块(module) | webpack 中文网 (webpackjs.com)](https://www.webpackjs.com/configuration/module/#rule-loaders) 发现`Rule.loaders`已弃用，改为`Rule.use`

> ## `Rule.loaders`
>
> > 由于需要支持 `Rule.use`，此选项**已废弃**。
>
> `Rule.loaders` 是 `Rule.use` 的别名。详细请查看 [`Rule.use`](https://www.webpackjs.com/configuration/module/#rule-use)。