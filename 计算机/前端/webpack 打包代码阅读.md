# webpack 打包代码阅读

以一个简单的 vue 项目为例

## 执行路径

`public/index.html` 中的 `index.html` 导入 `js/app.js` 文件

```html
<script defer="defer" src="/js/app.0920ac98.js"></script>
```

在 `js/app.js` 中最外层是一个 IIFE（立即执行函数）

```javascript
(function () {
  /* 暂时忽略 */
})()
```

其中定义了`__webpack_modules__`  `__webpack_require__` `__webpack_exports__` 等变量

此外，其中还通过 IIFE 为 `__webpack_require__` 添加了一些属性，但是并没有执行一些有影响的代码内容

顶级 IIFE 中执行的实际代码仅有一行

```javascript
var __webpack_exports__ = __webpack_require__(32225)
```

此代码实际调用 `__webpack_modules__[33225]` 的函数

```javascript
var __webpack_modules__ = {
  32225: function (__unused_webpack_module, __unused_webpack_exports, __webpack_require__) {
    Promise.all(/* import() */ [__webpack_require__.e(474), __webpack_require__.e(76)]).then(
      __webpack_require__.bind(__webpack_require__, 61076)
    )
  },

  32966: function (module, __unused_webpack_exports, __webpack_require__) {
    /* 暂时忽略 */
  },
}
```



## 顶级变量

### \_\_webpack\_modules\_\_

为各个模块的 Map

```javascript
var __webpack_modules__ = {
  32225: function (__unused_webpack_module, __unused_webpack_exports, __webpack_require__) {
    Promise.all(/* import() */ [__webpack_require__.e(474), __webpack_require__.e(76)]).then(
      __webpack_require__.bind(__webpack_require__, 61076)
    )
  },

  32966: function (module, __unused_webpack_exports, __webpack_require__) {
    'use strict'
    var __webpack_error__ = new Error()
    module.exports = new Promise(function (resolve, reject) {
      if (typeof provider !== 'undefined') return resolve()
      __webpack_require__.l(
        '/provider/remoteEntry.js',
        function (event) {
          if (typeof provider !== 'undefined') return resolve()
          var errorType = event && (event.type === 'load' ? 'missing' : event.type)
          var realSrc = event && event.target && event.target.src
          __webpack_error__.message =
            'Loading script failed.\n(' + errorType + ': ' + realSrc + ')'
          __webpack_error__.name = 'ScriptExternalLoadError'
          __webpack_error__.type = errorType
          __webpack_error__.request = realSrc
          reject(__webpack_error__)
        },
        'provider'
      )
    }).then(function () {
      return provider
    })
  },
}
```

### \_\_webpack\_module\_cache\_\_

用于缓存 `__webpack_modules__`

## \_\_webpack\_require\_\_

#### `__webpack_require__` 

是一个函数，用于导入模块

```javascript
function __webpack_require__(moduleId) {
  // 从缓存中进行读取，如果缓存中存在该模块，则直接返回
  var cachedModule = __webpack_module_cache__[moduleId]
  if (cachedModule !== undefined) {
    return cachedModule.exports
  }
  // 创建一个 module 对象，并放入缓存当中
  // Create a new module (and put it into the cache)
  var module = (__webpack_module_cache__[moduleId] = {
    id: moduleId,
    loaded: false,
    exports: {},
  })
  // 执行 __webpack_modules__ 中对应的函数
  // Execute the module function
  __webpack_modules__[moduleId].call(module.exports, module, module.exports, __webpack_require__)
  // 标识模块已经导入
  // Flag the module as loaded
  module.loaded = true
	// 返回模块导出
  // Return the exports of the module
  return module.exports
}
```

#### `__webpack_require__.c`

对象，等价于 `__webpack_module_cache__`

```javascript
// expose the module cache
__webpack_require__.c = __webpack_module_cache__
```

#### `__webpack_require__.m` 

对象，等价于 `__webpack_modules__`

```javascript
// expose the modules object (__webpack_modules__)
__webpack_require__.m = __webpack_modules__
```

#### `__webpack_require__.amdO`

对象

```javascript
/* webpack/runtime/amd options */
!(function () {
  __webpack_require__.amdO = {}
})()
```

