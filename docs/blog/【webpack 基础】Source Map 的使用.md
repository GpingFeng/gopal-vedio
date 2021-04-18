# 【webpack】聊聊 Source Map 的使用

## 前言

本文主要聊聊为什么要在 Webpack 中使用 Source Map？以及 Webpack 提供了哪些 Source Map 的使用方式，我们应该在开发环境和生产环境如何使用 Source map

本文使用的 Webpack 版本是 5.25.1，[Demo Github 地址](https://github.com/GpingFeng/leaarn-source-map/tree/main)

## Webpack 打包出来的代码有什么问题？

我们知道 Webpack 通过模块之间的引用关系，构建一个依赖树，并生成相应的结果文件。但这个结果文件是存在一定的缺陷的

- 代码有可能压缩并混淆
- 代码文件可能是由一个或者多个组成

以上两个问题就会导致：**假如你的代码报错，你该如何去定位问题？**

比如我在入口文件中：

```js
console.log('Interesting!!!')
// Create heading node
const heading = document.createElement('h1')
heading.textContent = 'Interesting!'
console.log(a); // 这一行会报错
// Append heading node to the DOM
const app = document.querySelector('#root')
app.append(heading)
```



![](https://tva1.sinaimg.cn/large/008eGmZEgy1gpmkstnbplj30yg0a2q57.jpg)



点进去 `source` 中，你可能是一脸懵逼的，因为代码是压缩混淆的，你根本不知道哪里报错了。这就是我们需要 Source Map 的重要原因

![ ](https://tva1.sinaimg.cn/large/008eGmZEgy1gp8pki7lxqj30yg0khwpw.jpg)

## 什么是 Source Map

**`Source Map`， 顾名思义，是保存源代码映射关系的文件**。上面提到的，我们找不到报错的文件的相关信息，那有没有一个拥有源文件与打包后文件的映射关系的文件，让它来告诉我们呢？它就是 Source Map 文件

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gp8pkplpa0j30k60ccdfs.jpg)



## 如何使用 Source Map

假如我们有了 Source Map 文件，我们如何使用它呢？我们只需要在打包后的文件的末尾加上：

```js
//# sourceMappingURL=main.bundle.js.map
```

sourceMappingURL 指向 `Source Map` 文件的 `URL`



## Source Map 文件解析

Source Map 文件大致如下所示：

```js
{
  "version": 3,
  "sources": [
    "webpack://webpack5-template/./src/index.js"
  ],
  "names": [],
  "mappings": ";;;;;AAAA;AACA;AACA;AACA;AACA,eAAe;AACf;AACA;AACA,mB",
  "file": "main.bundle.js",
  "sourcesContent": [
    "console.log('Interesting!!!')\n// Create heading node\nconst heading = document.createElement('h1')\nheading.textContent = 'Interesting!'\nconsole.log(a); // 这一行会报错\n// Append heading node to the DOM\nconst app = document.querySelector('#root')\napp.append(heading)"
  ],
  "sourceRoot": ""
}
```

　　- version：Source map 的版本，目前为 3。
　　- sources：转换前的文件。该项是一个数组，表示可能存在多个文件合并。
　　- names：转换前的所有变量名和属性名。
　　- mappings：记录位置信息的字符串。这个的话，用到了 VLQ 编码相关，详细可以看阮一峰老师的 [JavaScript Source Map 详解](https://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)
　　- file：转换后的文件名。
　　- sourceRoot：转换前的文件所在的目录。如果与转换前的文件在同一目录，该项为空。

## Webpack 中 Source Map

了解了 `Source Map` 的一些基础概念后，我们来看看在 Webpack 是如何使用 Source Map

我们先来看看 Webpack 中的 [devtool](https://webpack.docschina.org/configuration/devtool/) 配置

官方文档列出了很多种组合，在这之前，我们可以先好好看看以下的关键字，不管是什么组合都是下面的一个或者多个拼接而成的

- source map。产生 .map 文件（配合 eval 或者 inline 使用的时候，会不生成 source map 文件，具体要看哪个模式）
- eval。使用 eval 包裹块代码
- cheap。不生成列信息
- inline。将 .map 作为 DataURI 嵌入，不单独生成一个 .map 文件
- module。包含 loader 的 source map

接下来，我们用几个实例讲解一下

### devtool: 'source-map'

打包出来的 `main.bundle.js` ，可以看到最后一行是 `//# sourceMappingURL=main.bundle.js.map`，就是告诉浏览器源码所在的位置 是 `main.bundle.js.map`

```js
/******/ (() => { // webpackBootstrap
var __webpack_exports__ = {};
console.log('Interesting!!!')
// Create heading node
const heading = document.createElement('h1')
heading.textContent = 'Interesting!'
console.log(a); // 这一行会报错
// Append heading node to the DOM
const app = document.querySelector('#root')
app.append(heading)
/******/ })()
;
//# sourceMappingURL=main.bundle.js.map
```

然后同级目录下 `main.bundle.js.map`，是比较详细的

```js
{
  "version": 3,
  "sources": [
    "webpack://webpack5-template/./src/index.js"
  ],
  "names": [],
  "mappings": ";;;;;AAAA;AACA;AACA;AACA;AACA,eAAe;AACf;AACA;AACA,mB",
  "file": "main.bundle.js",
  "sourcesContent": [
    "console.log('Interesting!!!')\n// Create heading node\nconst heading = document.createElement('h1')\nheading.textContent = 'Interesting!'\nconsole.log(a); // 这一行会报错\n// Append heading node to the DOM\nconst app = document.querySelector('#root')\napp.append(heading)"
  ],
  "sourceRoot": ""
}
```

经过打包之后，重新查看我们的页面，是可以看到具体的报错的行数和列数（可以具体定位到某一列）

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gpml3upk0mj30pu05g3z5.jpg)

![image-20210417113405705](【webpack 基础】Source Map 的使用.assets/image-20210417113405705.png)

### devtool: 'cheap-source-map'

 yarn build 打包后，我们发现 mapping  部分不一样。主要是因为 cheap 不生成列信息，所以会少一些。我们测试的代码比较少，所以看起来区别不大，但如果代码量很大的时候，实际上会差别挺大的。具体的表现的话，跟上面有点差不多，就是点进去详情的时候，光标不会自动跳到具体某一列

```diff
{
  "version": 3,
  "file": "main.bundle.js",
  "sources": [
    "webpack://webpack5-template/./src/index.js"
  ],
  "sourcesContent": [
    "console.log('Interesting!!!')\n// Create heading node\nconst heading = document.createElement('h1')\nheading.textContent = 'Interesting!'\nconsole.log(a); // 这一行会报错\n// Append heading node to the DOM\nconst app = document.querySelector('#root')\napp.append(heading)"
  ],
-+  "mappings": ";;;;;AAAA;AACA;AACA;AACA;AACA;AACA;AACA;AACA;;A",
  "sourceRoot": ""
}

```

### devtool: 'cheap-module-source-map'

生成一个没有列信息（column-mappings）的 SourceMaps 文件，同时 loader 的 sourcemap 也被简化为只包含对应行的。

```diff
{
  "version": 3,
  "file": "main.bundle.js",
  "sources": [
    "webpack://webpack5-template/./src/index.js"
  ],
  "sourcesContent": [
    "console.log('Interesting!!!')\n// Create heading node\nconst heading = document.createElement('h1')\nheading.textContent = 'Interesting!'\nconsole.log(a); // 这一行会报错\n// Append heading node to the DOM\nconst app = document.querySelector('#root')\napp.append(heading)"
  ],
-+  "mappings": ";;;;;AAAA;AACA;AACA;AACA;AACA;AACA;AACA;AAAA;AACA;;A",
  "sourceRoot": ""
}
```



### devtool: 'eval-source-map'

打包出来只有 `main.bundle.js`。`eval-source-map` - 每个模块使用 `eval()` 执行，并且 `source map` 转换为 `DataUrl` 后添加到 `eval()` 中。这个模式跟 source map 放在一个全新的文件中，演示效果不再重复

大致的代码如下：

```js

(() => { // webpackBootstrap
	var __webpack_modules__ = ({
/***/ "./src/index.js":
/***/ (() => {
eval("console.log('Interesting!!!'); // Create heading node\n\nvar heading = document.createElement('h1');\nheading.textContent = 'Interesting!';\nconsole.log(a); // 这一行会报错\n// Append heading node to the DOM\n\nvar app = document.querySelector('#root');\napp.append(heading);//# sourceURL=[module]\n//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly93ZWJwYWNrNS10ZW1wbGF0ZS8uL3NyYy9pbmRleC5qcz9iNjM1Il0sIm5hbWVzIjpbImNvbnNvbGUiLCJsb2ciLCJoZWFkaW5nIiwiZG9jdW1lbnQiLCJjcmVhdGVFbGVtZW50IiwidGV4dENvbnRlbnQiLCJhIiwiYXBwIiwicXVlcnlTZWxlY3RvciIsImFwcGVuZCJdLCJtYXBwaW5ncyI6IkFBQUFBLE9BQU8sQ0FBQ0MsR0FBUixDQUFZLGdCQUFaLEUsQ0FDQTs7QUFDQSxJQUFNQyxPQUFPLEdBQUdDLFFBQVEsQ0FBQ0MsYUFBVCxDQUF1QixJQUF2QixDQUFoQjtBQUNBRixPQUFPLENBQUNHLFdBQVIsR0FBc0IsY0FBdEI7QUFDQUwsT0FBTyxDQUFDQyxHQUFSLENBQVlLLENBQVosRSxDQUFnQjtBQUNoQjs7QUFDQSxJQUFNQyxHQUFHLEdBQUdKLFFBQVEsQ0FBQ0ssYUFBVCxDQUF1QixPQUF2QixDQUFaO0FBQ0FELEdBQUcsQ0FBQ0UsTUFBSixDQUFXUCxPQUFYIiwic291cmNlc0NvbnRlbnQiOlsiY29uc29sZS5sb2coJ0ludGVyZXN0aW5nISEhJylcbi8vIENyZWF0ZSBoZWFkaW5nIG5vZGVcbmNvbnN0IGhlYWRpbmcgPSBkb2N1bWVudC5jcmVhdGVFbGVtZW50KCdoMScpXG5oZWFkaW5nLnRleHRDb250ZW50ID0gJ0ludGVyZXN0aW5nISdcbmNvbnNvbGUubG9nKGEpOyAvLyDov5nkuIDooYzkvJrmiqXplJlcbi8vIEFwcGVuZCBoZWFkaW5nIG5vZGUgdG8gdGhlIERPTVxuY29uc3QgYXBwID0gZG9jdW1lbnQucXVlcnlTZWxlY3RvcignI3Jvb3QnKVxuYXBwLmFwcGVuZChoZWFkaW5nKSJdLCJmaWxlIjoiLi9zcmMvaW5kZXguanMuanMiLCJzb3VyY2VSb290IjoiIn0=\n//# sourceURL=webpack-internal:///./src/index.js\n");
/***/ })
	});
	var __webpack_exports__ = {};
	__webpack_modules__["./src/index.js"]();
	
})()
;
```

### devtool: 'inline-source-map'

我们看到实际上只打包出来 `main.bundle.js` 文件，没有 `source map` ，这个时候实际上 `Souce Map` 实际上是内嵌到我们的 `main.bundle.js` 中了（留意 diff 的那一行）

```diff
/******/ (() => { // webpackBootstrap
var __webpack_exports__ = {};
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
console.log('Interesting!!!')
// Create heading node
const heading = document.createElement('h1')
heading.textContent = 'Interesting!'
console.log(a); // 这一行会报错
// Append heading node to the DOM
const app = document.querySelector('#root')
app.append(heading)
/******/ })()
;
+ //# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly93ZWJwYWNrNS10ZW1wbGF0ZS8uL3NyYy9pbmRleC5qcyJdLCJuYW1lcyI6W10sIm1hcHBpbmdzIjoiOzs7OztBQUFBO0FBQ0E7QUFDQTtBQUNBO0FBQ0EsZUFBZTtBQUNmO0FBQ0E7QUFDQSxtQiIsImZpbGUiOiJtYWluLmJ1bmRsZS5qcyIsInNvdXJjZXNDb250ZW50IjpbImNvbnNvbGUubG9nKCdJbnRlcmVzdGluZyEhIScpXG4vLyBDcmVhdGUgaGVhZGluZyBub2RlXG5jb25zdCBoZWFkaW5nID0gZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgnaDEnKVxuaGVhZGluZy50ZXh0Q29udGVudCA9ICdJbnRlcmVzdGluZyEnXG5jb25zb2xlLmxvZyhhKTsgLy8g6L+Z5LiA6KGM5Lya5oql6ZSZXG4vLyBBcHBlbmQgaGVhZGluZyBub2RlIHRvIHRoZSBET01cbmNvbnN0IGFwcCA9IGRvY3VtZW50LnF1ZXJ5U2VsZWN0b3IoJyNyb290JylcbmFwcC5hcHBlbmQoaGVhZGluZykiXSwic291cmNlUm9vdCI6IiJ9
```

### 小结

以上只是一些常见的模式的分析，官方文档给出很多的[模式](https://webpack.docschina.org/configuration/devtool/)

但基本都符合如下的结论

- source map。产生 .map 文件（配合 eval 或者 inline 使用的时候，会不生成 source map 文件，具体要看哪个模式）
- eval。使用 eval 包裹块代码
- cheap。不生成列信息
- inline。将 .map 作为 DataURI 嵌入，不单独生成一个 .map 文件
- module。包含 loader 的 source map



## 开发环境和生产环境

我们在开发环境和生产环境应该使用哪些模式?

### 开发环境

对于开发环境，通常希望更快速的 source map，需要添加到 bundle 中以增加体积为代价，但是对于生产环境，则希望更精准的 source map，需要从 bundle 中分离并独立存在。

对于开发环境，`eval`, `eval-source-map`, `eval-cheap-source-map`, `eval-cheap-module-source-map` 等都是可以的。个人推荐使用 `eval-cheap-module-source-map`

- `eval` 的执行效率高
- 这是 "cheap (低开销)" 的 source map，因为它没有生成列映射 (column mapping)，只是映射行数
- 源自 loader 的 source map 会得到更好的处理结果

### 生产环境

对于生产环境，一般选择`(none)`（省略 `devtool` 选项） - 不生成 source map。这是一个不错的选择。

一个特殊的场景，你需要在生产环境使用到 source map —— 监控系统分析具体错误信息，这个时候一般选择 `source-map`——整个 source map 作为一个单独的文件生成（当然如果你不需要获取列信息，可以使用 cheap-module-source-map）。它为 bundle 添加了一个引用注释，以便开发工具知道在哪里可以找到它。但需要注意的是要**将你的服务器配置为，不允许普通用户访问 source map 文件！你不应将 source map 文件部署到 web 服务器。而是只将其用于错误报告工具。**

### 监控系统分析具体错误信息实现

大致总结下它的工作流程，不再展开

- webpack 构建的时候将原始 JS 和 source map 文件都上传到我们的监控平台
- js 错误堆栈收集，通过 window.onerror 来捕获 js 报错，然后上报到服务器，以用来收集用户使用时候发生的 bug
- 解析 JS 错误，映射源文件堆栈
- 通过 sourcemap 查找原始报错信息，可以使用[source-map](https://github.com/mozilla/source-map)
- 监控平台展示

## 总结

因为 Webpack 打包会将代码混淆和压缩等，所以我们需要 Source Map 给我们解析出源文件，方便我们定位查看问题。Webpack 提供了很多种 devtool 的配置，但我们需要掌握`source map`、`eval`、`cheap`、`inline`、`module` 的大致具体含义，这样我们能够举一反三。对于生产环境和开发环境，我们需要采取不同的 `source map`策略，开发环境注重开发效率，生产环境则注重性能和安全。



[Demo Github 地址](https://github.com/GpingFeng/leaarn-source-map/tree/main)，希望对大家有所帮助，欢迎大家点赞评论收藏



## 参考

- https://juejin.cn/post/6844903450644316174
- https://segmentfault.com/a/1190000008315937