前几天面试的时候面试官问了两个问题：

- `axios` 底层封装的是什么？
- 什么情况下 `axios` 会报错？

在此之前只看过 `axios` 底层 `Promise` 的自然 G 了，面试完就开始看 `axios` 的源代码。

> 本文基于 `axios v1.1.3`

## 底层封装

首先来看看 `axios` 底层到底封装的什么

`axios` 底层封装的请求有两种：`XMLHttpRequest` 和 `Node.js` 的 `http/https`，位于 `lib/adapters` 文件夹内。接下来我们以 `XMLHttpRequest` 的封装为例看下什么情况下会报错

## 报错情况

`XMLHttpRequest` 的封装在 `lib/adapters/xhr.js` 文件夹内的 `xhrAdapter`，该函数直接返回一个 `Promise`

```javascript
export default function xhrAdapter(config) {
    return new Promise(function dispatchXhrRequest(resolve, reject) {
        ...
    });
```

与报错相关的逻辑如下

```javascript
export default function xhrAdapter(config) {
    return new Promise(function dispatchXhrRequest(resolve, reject) {
        ...
        // 初始化一个 XMLHttpRequest 对象
        let request = new XMLHttpRequest();
        ...
        // 初始化一个新创建的请求
        request.open(config.method.toUpperCase(), buildURL(fullPath, config.params, config.paramsSerializer), true);
        ...
        // 用于处理请求成功时的情况
        function onloadend() {
            ...
            settle(function _resolve(value) {
                resolve(value);
                done();
            }, function _reject(err) {
                reject(err);
                done();
            }, response);
            ...
        }
        
        // 请求成功时的回调
        if ('onloadend' in request) {
            // Use onloadend if available
            request.onloadend = onloadend;
        } else {
            // Listen for ready state to emulate onloadend
            request.onreadystatechange = function handleLoad() {
                if (!request || request.readyState !== 4) {
                    return;
                }

                if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
                    return;
                }
                setTimeout(onloadend);
            };
        }
            
        // 处理取消请求
        request.onabort = function handleAbort() {
            if (!request) {
                return;
            }

            reject(new AxiosError('Request aborted', AxiosError.ECONNABORTED, config, request));
            ...
        };
        // 处理网络错误
        request.onerror = function handleError() {
            reject(new AxiosError('Network Error', AxiosError.ERR_NETWORK, config, request));
            ...
        };
        // 处理超时
        request.ontimeout = function handleTimeout() {
            ...
            reject(new AxiosError(
                timeoutErrorMessage,
                transitional.clarifyTimeoutError ? AxiosError.ETIMEDOUT : AxiosError.ECONNABORTED,
                  config,
                request));
            ...
        };
        // 发送请求
        request.send(requestData || null);
    });
}
```

可以看到 `onabort` `onerror` `ontimeout` 三个回调都是直接处理一下然后调用 `reject`，但是 `onloadend` 并不是这样，我们来仔细看一下这一部分

```javascript
        // 用于处理请求成功时的情况
        function onloadend() {
            ...
            settle(function _resolve(value) {
                resolve(value);
                done();
            }, function _reject(err) {
                reject(err);
                done();
            }, response);
            ...
        }
        
        // 请求成功时的回调
        if ('onloadend' in request) {
            // Use onloadend if available
            request.onloadend = onloadend;
        } else {
            // Listen for ready state to emulate onloadend
            request.onreadystatechange = function handleLoad() {
                if (!request || request.readyState !== 4) {
                    return;
                }

                if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
                    return;
                }
                setTimeout(onloadend);
            };
        }
```

源代码中先是定义了一个 `onloadend` 函数，接下来判断 `request` 对象中是否有 `onloadend` 属性，个人猜测可能是为了兼容旧版本的浏览器。如果不存在这个属性，则走另一种可能，这个待会再说。继续看 `onloadend` 函数，里面的核心就是调用 `settle` 函数，此函数位于 `lib/core/settle.js`。

```javascript
export default function settle(resolve, reject, response) {
    const validateStatus = response.config.validateStatus;
    if (!response.status || !validateStatus || validateStatus(response.status)) {
        resolve(response);
    } else {
        reject(new AxiosError(
            'Request failed with status code ' + response.status,
            [AxiosError.ERR_BAD_REQUEST, AxiosError.ERR_BAD_RESPONSE][Math.floor(response.status / 100) - 4],
            response.config,
            response.request,
            response
        ));
    }
}
```

可以看到 `if (!response.status || !validateStatus || validateStatus(response.status))`，即当以下三个条件满足其一时即可 `resolve`

- 服务器返回的响应中不存在状态码或状态码为 0
- `axios` 的请求配置中 `validateStatus` 属性为空
- `validateStatus` 函数返回 `true`

否则则 `reject`。那么 `validateStatus` 又是什么？我们来看 `axios` 的默认请求配置 `default`，位于 `lib/defaults/index.js` 文件内

```javascript
const defaults = {
    validateStatus: function validateStatus(status) {
        return status >= 200 && status < 300;
    },
};
```

可以看到，默认的 `validateStatus` 是一个函数，当状态码在 [200,300) 时返回 `true`，否则返回 `false`。

在 `XMLHttpRequest` 对象触发事件的顺序中 `onabort` `ontimeout` `onerror` 在 `onloadend` 之前，所以如果发生了异常，则在 `onloadend` 执行之前已经调用了 `reject`，`onloadend` 中即使调用 `resolve` 也没有作用了。如果请求成功则由 `onloadend` 进行处理。

接下来我们来看 `XMLHttpRequest` 对象中没有 `onloadend` 属性的时候

```javascript
        // 请求成功时的回调
        if ('onloadend' in request) {
            // Use onloadend if available
            request.onloadend = onloadend;
        } else {
            // Listen for ready state to emulate onloadend
            request.onreadystatechange = function handleLoad() {
                if (!request || request.readyState !== 4) {
                    return;
                }

                if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
                    return;
                }
                setTimeout(onloadend);
            };
        }
```

根据源代码中的注释可以知道此处通过 `onreadystatechange` 来模拟 `onloadend`。在这里 `readyState` 不为 4 则直接返回，不进行任何操作。`readyState` 为 4 代表着请求已经结束了，无论成功还是失败。`if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0))` 是因为如果使用 `file:` 协议，则即使成功请求，状态码也会是 0，所以如果状态码是 0 并且不是走的 `file:` 协议则直接返回，接下来将 `onloadend` 加入到宏任务队列中去。

![image-20221029182249517](axios%20%E5%BA%95%E5%B1%82%E6%98%AF%E5%A6%82%E4%BD%95%E5%88%A4%E6%96%AD%E6%98%AF%E5%90%A6%E5%BA%94%E8%AF%A5%E6%8A%A5%E9%94%99%E7%9A%84.assets/image-20221029182249517.png)

![image-20221029182215984](axios%20%E5%BA%95%E5%B1%82%E6%98%AF%E5%A6%82%E4%BD%95%E5%88%A4%E6%96%AD%E6%98%AF%E5%90%A6%E5%BA%94%E8%AF%A5%E6%8A%A5%E9%94%99%E7%9A%84.assets/image-20221029182215984.png)

通过控制台模拟可知模拟的 `onloadend` 发生在 `onloadend` 之后

## 总结

`axios` 默认底层封装了 `XMLHttpRequest` 和 `http/https` 库，默认情况下只有以下四种情况会报错

- 取消请求
- 网络错误
- 超时
- 状态码不在 [200,300) 区间内