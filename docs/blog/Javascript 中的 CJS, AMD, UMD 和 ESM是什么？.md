# Javascript 中的 CJS, AMD, UMD 和 ESM是什么？



原文地址：[What are CJS, AMD, UMD, and ESM in Javascript?](https://dev.to/iggredible/what-the-heck-are-cjs-amd-umd-and-esm-ikm)

原文作者：[Igor Irianto](https://dev.to/iggredible)

译者：`Gopal`

最初，`Javascript` 没有导入/导出模块的方法， 这是让人头疼的问题。 想象一下，只用一个文件编写应用程序——这简直是噩梦！

然后，很多比我聪明得多的人试图给 `Javascript` 添加模块化。其中就有 `CJS`、`AMD`、`UMD` 和 `ESM`。你可能听说过其中的一些方法(还有其他方法，但这些是比较通用的)。

我将介绍它们：它们的语法、目的和基本行为。我的目标是帮助读者在看到它们时认出它们

## CJS

`CJS` 是 `CommonJS` 的缩写。经常我们这么使用：

```javascript
// importing 
const doSomething = require('./doSomething.js'); 

// exporting
module.exports = function doSomething(n) {
  // do something
}
```



- 很多人可以从 `Node` 中立刻认出 `CJS` 的语法。这是因为 `Node` 就是使用 [`CJS` 模块](https://blog.risingstack.com/node-js-at-scale-module-system-commonjs-require/)的
- `CJS` 是同步导入模块
- 你可以从 `node_modules` 中引入一个库或者从本地目录引入一个文件 。如 `const myLocalModule = require('./some/local/file.js')` 或者 `var React = require('react');` ，都可以起作用
- 当 `CJS` 导入时，它会给你一个导入对象的副本
- `CJS` 不能在浏览器中工作。它必须经过转换和打包

## AMD

`AMD` 代表异步模块定义。下面是一个示例代码

```javascript
define(['dep1', 'dep2'], function (dep1, dep2) {
    //Define the module value by returning a value.
    return function () {};
});
```

或者

```javascript
// "simplified CommonJS wrapping" https://requirejs.org/docs/whyamd.html
define(function (require) {
    var dep1 = require('dep1'),
        dep2 = require('dep2');
    return function () {};
});
```

- `AMD` 是异步(`asynchronously`)导入模块的(因此得名) 
- 一开始被提议的时候，`AMD` 是为前端而做的(而 `CJS` 是后端)
- `AMD` 的语法不如 `CJS` 直观。我认为 `AMD` 和 `CJS` 完全相反

## UMD

`UMD` 代表通用模块定义（`Universal Module Definition`）。下面是它可能的样子([来源](http://bob.yexley.net/umd-javascript-that-runs-anywhere/))

```javascript
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["jquery", "underscore"], factory);
    } else if (typeof exports === "object") {
        module.exports = factory(require("jquery"), require("underscore"));
    } else {
        root.Requester = factory(root.$, root._);
    }
}(this, function ($, _) {
    // this is where I defined my module implementation

    var Requester = { // ... };

    return Requester;
}));
```

- 在前端和后端都适用（“通用”因此得名）
- 与 `CJS` 或 `AMD` `不同，UMD` 更像是一种配置多个模块系统的模式。[这里](https://github.com/umdjs/umd/)可以找到更多的模式
- 当使用 `Rollup/Webpack` 之类的打包器时，`UMD` 通常用作备用模块



## ESM 

`ESM` 代表 `ES` 模块。这是 `Javascript` 提出的实现一个标准模块系统的方案。我相信你们很多人都看到过这个:

```javascript
import React from 'react';
```

或者其他更多的

```javascript
import {foo, bar} from './myLib';

...

export default function() {
  // your Function
};
export const function1() {...};
export const function2() {...};
```

- 在很多[现代浏览器](https://caniuse.com/es6-module)可以使用
- 它兼具两方面的优点：具有 `CJS` 的简单语法和 `AMD` 的异步
- 得益于 `ES6` 的[静态模块结构](https://exploringjs.com/es6/ch_modules.html#sec_design-goals-es6-modules)，可以进行 [ Tree Shaking](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/tree-shaking/)
- `ESM` 允许像 `Rollup` 这样的打包器，[删除不必要的代码](https://dev.to/bennypowers/you-should-be-using-esm-kn3)，减少代码包可以获得更快的加载
- 可以在 `HTML` 中调用，只要如下

```javascript
<script type="module">
  import {func1} from 'my-lib';

  func1();
</script>
```

但是不是所有的浏览器都支持（[来源](https://jakearchibald.com/2017/es-modules-in-browsers/)）

## 总结

- 由于 `ESM` 具有简单的语法，异步特性和可摇树性，因此它是最好的模块化方案
- `UMD` 随处可见，通常在 `ESM` 不起作用的情况下用作备用
- `CJS` 是同步的，适合后端
- `AMD` 是异步的，适合前端	

感谢你的阅读，开发者们！在未来，我计划深入讨论每个模块，特别是 ESM，因为它包含了许多很棒的东西。请继续关注!

如果你注意到任何错误，请告诉我。

## 参考

- [basic js modules](https://www.freecodecamp.org/news/anatomy-of-js-module-systems-and-building-libraries-fadcd8dbd0e/)
- [CJS in nodejs](https://blog.risingstack.com/node-js-at-scale-module-system-commonjs-require/)
- [CJs-ESM comparison](https://jsmodules.io/cjs.html)
- [On inventing JS module formats and script loaders](http://tagneto.blogspot.com/2011/04/on-inventing-js-module-formats-and.html)
- [Why use AMD](https://requirejs.org/docs/whyamd.html)
- [es6 modules browser compatibility](https://caniuse.com/#feat=es6-module)
- [Reduce JS payloads with tree-shaking](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/tree-shaking/)
- [JS modules - static structure](https://exploringjs.com/es6/ch_modules.html#static-module-structure)
- [ESM in browsers](https://jakearchibald.com/2017/es-modules-in-browsers/)
- [ES Modules deep dive - cartoon](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)
- [Reasons to use ESM](https://dev.to/bennypowers/you-should-be-using-esm-kn3)