

# 【webpack 进阶】聊聊 webpack 热更新以及原理

## 什么是热更新

模块热替换(`hot module replacement 或 HMR`)是 `webpack` 提供的最有用的功能之一。它允许在运行时更新所有类型的模块，而无需完全刷新

一般的刷新我们分两种：

- 一种是页面刷新，不保留页面状态，就是简单粗暴，直接 `window.location.reload()`。
- 另一种是基于 `WDS (Webpack-dev-server)` 的模块热替换，只需要局部刷新页面上发生变化的模块，同时可以保留当前的页面状态，比如复选框的选中状态、输入框的输入等。

可以看到相比于第一种，热更新对于我们的开发体验以及开发效率都具有重大的意义

`HMR ` 作为一个 `Webpack` 内置的功能，可以通过 `HotModuleReplacementPlugin` 或 `--hot` 开启。

具体我们如何在  `webpack`  中使用这个功能呢？

## 热更新的使用以及简单分析

### 如何使用热更新

```js
npm install webpack webpack-dev-server --save-dev
```

设置 `HotModuleReplacementPlugin`，`HotModuleReplacementPlugin` 是 `webpack`  是自带的

```js
plugins: {
    HotModuleReplacementPlugin: new webpack.HotModuleReplacementPlugin()
}
```

再设置一下 `devServer`

```js
devServer: {
    contentBase: path.resolve(__dirname, 'dist'),
    hot: true, // 重点关注
    historyApiFallback: true,
    compress: true
}
```

- `hot` 为 `true`，代表开启热更新

### 两个重要的文件

当我们改变我们项目的文件的时候，比如我修改 `Vue` 的一个 方法：

更改前：

```js
clickMe() {
  console.log('我是 Gopal，欢迎关注「前端杂货铺」');
}
```

更改后：

```js
clickMe() {
  console.log('我是 Gopal，欢迎关注「前端杂货铺」，一起学习成长吧');
}
```

浏览器会去请求两个文件 

![](https://upload-images.jianshu.io/upload_images/1784460-54005b7f50edf5f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来我们看看这两个文件：

- `JSON` 文件，`h `  代表本次新生成的 `Hash` 值为 `0c256052432b51ed32c8`——本次输出的 `Hash` 值会被作为下次热更新的标识。`c` 表示当前要热更新的文件对应的是哪个模块，可以让  `webpack`  知道它要更新哪个模块

```js
{
    "h": "0c256052432b51ed32c8",
    "c": {
        "201": true
    }
}
```

- `js` 文件，就是本次修改的代码，重新编译打包后的，大致是下面这个样子（已删减一些并格式化过，这里看不懂没关系的，就记住是返回要更新的模块就好了），`webpackHotUpdate` 方法就是用来更新模块的，`201` 对应的是哪个模块（我们称它为模块标识），其他的就是要更新的模块的内容了

```js
webpackHotUpdate(201, {
  "./src/views/moveTransfer/list/index.vue?vue&type=script&lang=js&": function (
    module,
    exports,
    __webpack_require__
  ) {
    "use strict";

    var _Object$defineProperty = __webpack_require__(
      /*! @babel/runtime-corejs3/core-js-stable/object/define-property */ "./node_modules/@babel/runtime-corejs3/core-js-stable/object/define-property.js"
    );

    _Object$defineProperty(exports, "__esModule", {
      value: true,
    });

    exports.default = void 0;

    var _default = {
      data: function data() {
        return {};
      },
      computed: {},
      methods: {
        clickMe: function clickMe() {
          console.log("我是 Gopal，欢迎关注「前端杂货铺」，一起学习成长吧");
        },
      },
    };
    exports.default = _default;
  },
});

```

那么问题来了，**我修改了文件，浏览器是怎么知道要更新的呢？**

## 了解一下 Websocket

热更新使用到了 `Websocket`，这里不会细讲 `Websocket`，可以看下阮一峰老师的  [WebSocket 教程](https://www.ruanyifeng.com/blog/2017/05/websocket.html)，下面是一个 [简单的例子](https://jsbin.com/muqamiqimu/edit?js,console)

```javascript
// 执行上面语句之后，客户端就会与服务器进行连接。
var ws = new WebSocket("wss://echo.websocket.org");

// 实例对象的 onopen 属性，用于指定连接成功后的回调函数
ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

// 实例对象的 onmessage 属性，用于指定收到服务器数据后的回调函数。可以接受二进制数据，blob 对象或者 Arraybuffer 对象
ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

// 实例对象的 onclose 属性，用于指定连接关闭后的回调函数。
ws.onclose = function(evt) {
  console.log("Connection closed.");
};      
```

上面通过 new Websocket 创建一个客户端与服务端通信的实例，并通过 `onmessage`属性，接受指定服务器返回的数据，并进行相应的处理。

这里大概解释下，为什么是 `Websocket` ？因为 `Websocket` 是一种双向协议，它最大的特点就是 **服务器可以主动向客户端推送消息，客户端也可以主动向服务器发送信息**。这是 `HTTP` 不具备的，**热更新实际上就是服务器端的更新通知到客户端，所以选择了 `Websocket`**

接下来让我们进一步的讨论关于热更新的原理

## 热更新原理

### 热更新的过程

几个重要的概念（这里有一个大致的概念就好，后面会把它们串起来）：

- `Webpack-complier` ：`webpack` 的编译器，将 `JavaScript` 编译成 `bundle`（就是最终的输出文件）
- `HMR Server`：将热更新的文件输出给 `HMR Runtime`
- `Bunble Server`：提供文件在浏览器的访问，也就是我们平时能够正常通过 `localhost` 访问我们本地网站的原因
- `HMR Runtime`：开启了热更新的话，在打包阶段会被注入到浏览器中的 `bundle.js`，这样 `bundle.js` 就可以跟服务器建立连接，通常是使用 `websocket` ，当收到服务器的更新指令的时候，就去更新文件的变化
- `bundle.js`：构建输出的文件

#### 启动阶段

文件经过 `Webpack-complier` 编译好后传输给 `Bundle Server`，`Bundle Server` 可以让浏览器访问到我们打包出来的文件

下面流程图中的 1、2、A、B阶段

#### 文件热更新阶段

文件经过 `Webpack-complier` 编译好后传输给 `HMR Server`，`HMR Server `知道哪个资源(模块)发生了改变，并通知 `HMR Runtime` 有哪些变化（也就是上面我们看到的两个请求），`HMR Runtime` 就会更新我们的代码，这样我们浏览器就会更新并且不需要刷新

下面流程图的 1、2、3、4、5 阶段

参考 [19 | webpack中的热更新及原理分析](https://time.geekbang.org/course/detail/100028901-98391)



![](https://upload-images.jianshu.io/upload_images/1784460-67579c84fb6fe1ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 深入——源码阅读

我们还看回上图，其中启动阶段图中的 1、2、A、B阶段就不讲解了，主要看热更新阶段主要讲 3、4 和 5 阶段

在开始接下开的阅读前，我们再回到最初的问题上**我本地修改了文件，浏览器是怎么知道要更新的呢？**

通过上面的流程图，其实我们可以猜测，**本地实际上启动了一个 `HMR Server` 服务，而且在启动 `Bundle Server` 的时候已经往我们的 `bundle.js` 中注入了 `HMR Runtime`(主要用来启动 `Websocket`，接受 `HMR Server` 发来的变更)**

所以我们聚焦放在以下几点：

- `Webpack` 如何启动了 HMR Server
- HMR Server 如何跟 HMR Runtime 进行通信的
- `HMR Runtime` 接受到变更之后，如何生效的

以下的源码解析分别对应的版本是：

- webpack——`5.24.3`
- webpack-dev-server——`4.0.0-beta.0`
- webpack-dev-middleware——`4.1.0`

### 启动 HMR Server

这个工作主要是在 `webpack-dev-server` 中完成的

看 `lib/Server.js` `setupApp` 方法，下面的 express 服务实际上对应的是 `Bundle Server`

```js
setupApp() {
  // Init express server
  // eslint-disable-next-line new-cap
  // 初始化 express 服务
  // 使用 express 框架启动本地 server，让浏览器可以请求本地的静态资源。
  this.app = new express();
}
```

启动服务结束之后就通过 `createSocketServer ` 创建 `websocket` 服务

```js
listen(port, hostname, fn) {
  this.hostname = hostname;
  return (
    findPort(port || this.options.port)
      .then((port) => {
        this.port = port;
        return this.server.listen(port, hostname, (err) => {
          if (this.options.hot || this.options.liveReload) {
            // 启动 express 服务之后，启动 websocket 服务
            this.createSocketServer();
          }
        });
      })
  );
}
```

```js
createSocketServer() {
  this.socketServer = new this.SocketServerImplementation(this);

  this.socketServer.onConnection((connection, headers) => {
	
  });
}
```

### HMR Server 和 HMR Runtime 的通信

首先要通信的第一个问题在于——**通信的时机，什么时候我去通知客户端我的文件更新**。通过 `webpack` 创建的 `compiler` 实例（监听本地文件的变化、文件改变自动编译、编译输出），可以往 `compiler.hooks.done` 钩子（代表 `webpack` 编译完之后触发）注册事件， 当监听到一次 webpack 编译结束，就会调用 sendStats 方法 

看 `lib/Server.js` 中的 `  setupHooks` 方法

```js
// lib/Server.js
// 绑定监听事件
setupHooks() {
  // ...
  const addHooks = (compiler) => {
    // 监听 webpack 的 done 钩子，tapable 提供的监听方法
    // done 标识编译结束
    const { compile, invalid, done } = compiler.hooks;
    compile.tap('webpack-dev-server', invalidPlugin);
    invalid.tap('webpack-dev-server', invalidPlugin);
    done.tap('webpack-dev-server', (stats) => {
      // 当监听到一次webpack编译结束，就会调用 sendStats 方法
      this.sendStats(this.sockets, this.getStats(stats));
      this.stats = stats;
    });
  };
}
```

当监听到一次 webpack 编译结束，就会调用 sendStats 方法，里面会向客户端发送 `hash` 和 `ok` 事件

```js
// lib/Server.js
// send stats to a socket or multiple sockets
sendStats(sockets, stats, force) {
  // ok和 hash
  this.sockWrite(sockets, 'hash', stats.hash);

  if (stats.errors.length > 0) {
    this.sockWrite(sockets, 'errors', stats.errors);
  } else if (stats.warnings.length > 0) {
    this.sockWrite(sockets, 'warnings', stats.warnings);
  } else {
    this.sockWrite(sockets, 'ok');
  }
}
```

在 `client-src/default/index.js` 中，会去更新 `hash`，并且在 `ok` 的时候去进行检查更新 `reloadApp`

```js
// client-src/default/index.js 
const onSocketMessage = {
  // 更新 current Hash
  hash(hash) {
    status.currentHash = hash;
  },
  'progress-update': function progressUpdate(data) {
    if (options.useProgress) {
      log.info(`${data.percent}% - ${data.msg}.`);
    }

    sendMessage('Progress', data);
  },
  ok() {
    sendMessage('Ok');

    if (options.useWarningOverlay || options.useErrorOverlay) {
      overlay.clear();
    }

    if (options.initial) {
      return (options.initial = false);
    }
    // 进行更新检查等操作
    reloadApp(options, status);
  }

};
```

 接下来我们看看 `client-src/default/utils/reloadApp.js` 中的 `reloadApp`。这里又利用 `node.js` 的 `EventEmitter`，发出`webpackHotUpdate` 消息。这里又将更新的事情给回了 `webpack`（为了更好的维护代码，以及职责划分的更明确。）

```js
function reloadApp(
  { hotReload, hot, liveReload },
  { isUnloading, currentHash }
) {
  // ...
  if (hot) {
    log.info('App hot update...');
    //  hotEmitter 其实就是 EventEmitter 的实例
    const hotEmitter = require('webpack/hot/emitter');
    // 又利用 node.js 的 EventEmitter，发出 webpackHotUpdate 消息。
    // websocket 仅仅用于客户端（浏览器）和服务端进行通信。而真正做事情的活还是交回给了 webpack。
    hotEmitter.emit('webpackHotUpdate', currentHash);
    if (typeof self !== 'undefined' && self.window) {
      // broadcast update to window
      self.postMessage(`webpackHotUpdate${currentHash}`, '*');
    }
  }
  // ...
}

module.exports = reloadApp;
```

在 `webpack `  的 `hot/dev-server.js` 中，监听 `webpackHotUpdate` 事件，并执行 `check` 方法。并在 `check` 方法中调用 `module.hot.check` 方法进行热更新。

```js
// hot/dev-server.js
// 监听webpackHotUpdate事件
hotEmitter.on("webpackHotUpdate", function (currentHash) {
  lastHash = currentHash;
  if (!upToDate() && module.hot.status() === "idle") {
    log("info", "[HMR] Checking for updates on the server...");
    check();
  }
});
```

```js
	var check = function check() {
		//  moudle.hot.check 开始热更新
		// 之后的源码都是HotModuleReplacementPlugin塞入到bundle.js中的哦，我就不写文件路径了
		module.hot
			.check(true)
			.then(function (updatedModules) {
				// ...
			})
			.catch(function (err) {
				// ...
			});
	};
```

至于 `module.hot.check` ，实际上通过 `HotModuleReplacementPlugin` 已经注入到我们 `chunk ` 中了（也就是我们上面所说的 HMR Runtime），所以后面就是它是如何更新 `bundle.js` 的呢？

### HMR Runtime 中更新 bundle.js

如果我们仔细看我们的打包后的文件的话，开启热更新之后生成的代码会比不开启多出很多东西（为了更加直观看到，可以将其输出到本地），这些就是帮助 `webpack` 在浏览器端去更新 `bundle.js` 的 `HMR Runtime` 代码

来看打包后的代码中新增了一个 `createModuleHotObject`

```js
module.hot = createModuleHotObject(options.id, module);
```

实际上这个函数就是用来返回一个 hot 对象，所以调用 `module.hot.check` 的时候，实际上就是执行 `hotCheck` 函数

```js
function createModuleHotObject(moduleId, me) {
	var hot = {
		// Module API
		addDisposeHandler: function (callback) {
			hot._disposeHandlers.push(callback);
		},
		removeDisposeHandler: function (callback) {
			var idx = hot._disposeHandlers.indexOf(callback);
			if (idx >= 0) hot._disposeHandlers.splice(idx, 1);
		},
		// Management API
		check: hotCheck,
		apply: hotApply,
		status: function (l) {
			if (!l) return currentStatus;
			registeredStatusHandlers.push(l);
		},
		addStatusHandler: function (l) {
			registeredStatusHandlers.push(l);
		},
		removeStatusHandler: function (l) {
			var idx = registeredStatusHandlers.indexOf(l);
			if (idx >= 0) registeredStatusHandlers.splice(idx, 1);
		},
	};
	currentChildModule = undefined;
	return hot;
}
```

其中就有 `hotCheck` 中调用了 `__webpack_require__.hmrM`

```js
function hotCheck(applyOnUpdate) {
	setStatus("check");
	return __webpack_require__.hmrM().then(function (update) {
  }
}
```

#### `__webpack_require__.hmrM`——加载.hot-update.json

来看 `__webpack_require__.hmrM`, 其中 `__webpack_require__.p` 指的是我们本地服务的域名，类似 `http://0.0.0.0:9528` ， 另外 `__webpack_require__.hmrF` 去获取 `.hot-update.json` 文件的地址，就是我们之前提到的重要文件之一

```js
__webpack_require__.hmrM = () => {
	if (typeof fetch === "undefined") throw new Error("No browser support: need fetch API");
	return fetch(__webpack_require__.p + __webpack_require__.hmrF()).then((response) => {
		if(response.status === 404) return; // no update available
		if(!response.ok) throw new Error("Failed to fetch update manifest " + response.statusText);
		return response.json();
	});
};
```

```js
/* webpack/runtime/get update manifest filename */
(() => {
  __webpack_require__.hmrF = () => ("main." + __webpack_require__.h() + ".hot-update.json");
})();
```

#### 加载要更新的模块

下面来看如何加载我们要更新的模块的，可以看到打包出来的代码中有 `loadUpdateChunk`

```js
function loadUpdateChunk(chunkId) {
	return new Promise((resolve, reject) => {
		var url = __webpack_require__.p + __webpack_require__.hu(chunkId);
		// create error before stack unwound to get useful stacktrace later
		var error = new Error();
		var loadingEnded = (event) => {
			// ...加载后的处理
		};
		__webpack_require__.l(url, loadingEnded);
	});
}
```

再来看 `__webpack_require__.l`，主要通过类似 `JSONP` 的方式进行，因为`JSONP`获取的代码可以直接执行。

```js
__webpack_require__.l = (url, done, key, chunkId) => {
  // ...
  if (!script) {
    script = document.createElement("script");

    script.charset = "utf-8";
    script.timeout = 120;
    if (__webpack_require__.nc) {
      script.setAttribute("nonce", __webpack_require__.nc);
    }
    script.setAttribute("data-webpack", dataWebpackPrefix + key);
    script.src = url;
  }
  // ...
  needAttach && document.head.appendChild(script);
};
```

还记得我们一开始提到的返回的 JS 中就是一个 `webpackHotUpdate` 函数么？实际上在我们的 HMR Runtime 中就是全局定义了（下面的名称是 webpackHotUpdatelearn_hot_reload，我猜想应该是 webpack 版本不一样导致的）至于生成的代码是如何生效的，请移步我的另外一篇文章——[【Webpack 进阶】Webpack 打包后的代码是怎样的？](https://juejin.cn/post/6937086236926410783)

```js
// webpackHotUpdate + 项目名
self["webpackHotUpdatelearn_hot_reload"] = (chunkId, moreModules, runtime) => {
	for(var moduleId in moreModules) {
		if(__webpack_require__.o(moreModules, moduleId)) {
			currentUpdate[moduleId] = moreModules[moduleId];
			if(currentUpdatedModulesList) currentUpdatedModulesList.push(moduleId);
		}
	}
	if(runtime) currentUpdateRuntime.push(runtime);
	if(waitingUpdateResolves[chunkId]) {
		waitingUpdateResolves[chunkId]();
		waitingUpdateResolves[chunkId] = undefined;
	}
};
```

所以，客户端接受到服务器端推动的消息后，如果需要热更新，**浏览器发起 http 请求去服务器端获取新的模块资源解析并局部刷新页面**

## 总结

本文介绍了 `webpack` 热更新的简单使用、相关的流程以及原理。小结一下，webpack 如果开启了热更新的时候

- `HMR Runtime` 通过 `HotModuleReplacementPlugin` 已经注入到我们 `chunk ` 中了

- 除了开启一个 ``Bundle Server``，还开启了 `HMR Server`，主要用来和 `HMR Runtime `中通信
- 在编译结束的时候，通过 `compiler.hooks.done`，监听并通知客户端
- 客户端接收到之后，就会调用  `module.hot.check` 等，发起 http 请求去服务器端获取新的模块资源解析并局部刷新页面

![](https://upload-images.jianshu.io/upload_images/1784460-67579c84fb6fe1ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考

-  [模块热替换(hot module replacement)]( https://webpack.docschina.org/concepts/hot-module-replacement/)
-  [轻松理解webpack热更新原理](https://juejin.cn/post/6844904008432222215)
-  [WebSocket 教程](https://www.ruanyifeng.com/blog/2017/05/websocket.html)
-  [搞懂webpack热更新原理](https://juejin.cn/post/6844903933157048333#heading-29) 
-  [看完这篇，再也不怕被问 Webpack 热更新](https://www.infoq.cn/article/dioufdrtt3rocojvlrcl)
-  [从零实现 webpack 热更新 HMR](https://time.geekbang.org/course/detail/100028901-98391)

