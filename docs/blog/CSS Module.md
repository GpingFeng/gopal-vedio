# 深入浅出 CSS Modules

## CSS Modules 是什么？

官方文档的介绍如下：

> A **CSS Modules** is a CSS file in which all class names and animation names are scoped locally by default.

所有的类名和动画名称默认都有各自的作用域的 `CSS` 文件。CSS Modules 并不是 CSS 官方的标准，也不是浏览器的特性，而是使用一些构建工具，比如 webpack，对 CSS 类名和选择器限定作用域的一种方式（类似命名空间）

本文来介绍一下 CSS Modules 的简单使用，以及 CSS Modules 的实现原理（CSS-loader 中的实现）

## CSS Modules 的简单使用

### 项目搭建以及配置

新建一个项目，本文的 [Demo](https://github.com/GpingFeng/learn-css-modules)

```bash
npx create-react-app learn-css-modules-react
cd learn-css-modules-react
yarn eject
```

看到 `config/webpack.config.js`，默认情况下，React 脚手架搭建出来的项目，只有 `.module.css` 支持模块化

```js
// Adds support for CSS Modules (https://github.com/css-modules/css-modules)
// using the extension .module.css
{
  test: cssModuleRegex,
  use: getStyleLoaders({
    importLoaders: 1,
    sourceMap: isEnvProduction
      ? shouldUseSourceMap
      : isEnvDevelopment,
    modules: {
      getLocalIdent: getCSSModuleLocalIdent,
    },
  }),
}
```

其中 getStyleLoaders 函数，可以看到配置 css-loader 

```js
  const getStyleLoaders = (cssOptions, preProcessor) => {
    const loaders = [
      // ...
      {
        loader: require.resolve('css-loader'),
        options: cssOptions,
      },
      // ...
    ];
    // ...
    return loaders;
  };
```

我们就基于这个环境当做示例

### 局部作用域

将 `App.css` 修改成 `App.module.css`

之前的样式

![](img/CSS Module/image-20210418121302565.png)

我们导入 css，并设置

```jsx
import styles from './App.module.css';

// ...
<header className={styles['App-header']}></header>
```

就会根据特定的规则生成相应的类名

![](img/CSS Module/image-20210418124516803.png)

这个命名规则是可以通过 CSS-loader 进行配置，类似如下的配置：

```javascript
module: {
  loaders: [
    // ...
    {
      test: /\.css$/,
      loader: "style-loader!css-loader?modules&localIdentName=[path][name]---[local]---[hash:base64:5]"
    },
  ]
}
```

### 全局作用域

我们发现，在 css modules 中定义的类名得通过类似设置变量的方式给 HTML 设置，那么我能像其他的 CSS 文件一样直接使用类名（也就是普通的设置方法），而不是编译后的哈希字符串么？

使用 `:global` 的语法，就可以声明一个全局的规则

```css
:global(.App-link) {
  color: #61dafb;
}
```

这样就可以在 HTML 中直接跟使用普通的 CSS 一样了

但这里感觉就是 CSS Modules 给开发者留的一个后门，我们这样的 CSS，还是直接放在普通 .css 文件中比较好（个人的不成熟的建议）

### Class 的组合

在 CSS Modules 中，一个选择器可以继承另一个选择器的规则，这称为 "组合"（["composition"](https://github.com/css-modules/css-modules#composition)）

比如，我们定义一个 font-red，然后在 .App-header 中使用 `composes: font-red;` 继承

```css
.font-red {
  color: red;
}

.App-header {
  composes: font-red;
  /* ... */
}
```



![](img/CSS Module/image-20210418125512741.png)

### 输入其他的模块

不仅仅可以同一个文件中的，还可以继承其他文件中的 CSS 规则

定义一个 `another.module.css`

```css
.font-blue {
  color: blue;
}
```

在 App.module.css 中

```css
.App-header {
  /* ... */
  composes: font-blue from './another.module.css';
  /* ... */
}
```



![](img/CSS Module/image-20210418125726167.png)

### 使用变量

我们还可以使用变量，定义一个 `colors.module.css`

```css
@value blue: #0c77f8;
```

在 App.module.css 中

```css
@value colors: "./colors.module.css";
@value blue from colors;

.App-header {
  /* ... */
  color: blue;
}
```



![](img/CSS Module/image-20210418130002303.png)



### 使用小结

总体而言，CSS Modules 的使用偏简单，上手非常的快，接下来我们看看 Webpack 中 CSS-loader 是怎么实现 CSS Modules 的

## CSS Modules 的实现原理

### 从 CSS Loader 开始讲起

看 `lib/processCss.js` 中

```js
var pipeline = postcss([
	...
	modulesValues,
	modulesScope({
    // 根据规则生成特定的名字
		generateScopedName: function(exportName) {
			return getLocalIdent(options.loaderContext, localIdentName, exportName, {
				regExp: localIdentRegExp,
				hashPrefix: query.hashPrefix || "",
				context: context
			});
		}
	}),
	parserPlugin(parserOptions)
]);
```

主要看 `modulesValues` 和 `modulesScope` 方法，实际上这两个方法又是来自其他两个包

```js
var modulesScope = require("postcss-modules-scope");
var modulesValues = require("postcss-modules-values");
```

### postcss-modules-scope

这个包主要是实现了 CSS Modules 的样式隔离（Scope Local）以及继承（Extend）

它的代码比较简单，基本一个文件完成，源码可以看[这里](https://github.com/css-modules/postcss-modules-scope/blob/master/src/index.js)，这里会用到 postcss 处理 AST 相关，我们大致了解它的思想即可

#### 默认的命名规则

实际上，假如你设置任何的规则时候会根据如下进行命名

```js
// 生成 Scoped name 的方法（没有传入的时候的默认规则）
processor.generateScopedName = function (exportedName, path) {
  var sanitisedPath = path.replace(/\.[^\.\/\\]+$/, '').replace(/[\W_]+/g, '_').replace(/^_|_$/g, '');
  return '_' + sanitisedPath + '__' + exportedName;
};
```

这种写法在很多的源码中我们都可以看到，以后写代码的时候也可以采用

```js
var processor = _postcss2['default'].plugin('postcss-modules-scope', function (options) {
  // ...
  return function (css) {
    // 如果有传入，则采用传入的命名规则
    var generateScopedName = options && options.generateScopedName || processor.generateScopedName;
  }
  // ...
})
```

#### 前置知识—— postcss 遍历样式的方法

css ast 主要有 3 种父类型

- AtRule: @xxx 的这种类型，如 @screen
- Comment: 注释
- Rule: 普通的 css 规则

还有几个个比较重要的子类型：

- decl： 指的是每条具体的 css 规则
- rule：作用于某个选择器上的 css 规则集合

不同的类型进行不同的遍历

- walk: 遍历所有节点信息，无论是 atRule、rule、comment 的父类型，还是 `rule`、 `decl` 的子类型
- walkAtRules：遍历所有的 atRule
- walkComments：遍历注释
- walkDecls
- walkRules

#### 作用域样式的实现

```js
// Find any :local classes
// 找到 :local 的 classes
css.walkRules(function (rule) {
  var selector = _cssSelectorTokenizer2['default'].parse(rule.selector);
  // 递归获取 selector
  var newSelector = traverseNode(selector);
  rule.selector = _cssSelectorTokenizer2['default'].stringify(newSelector);
  // 遍历每一条规则，假如匹配到则转换成作用域名称
  rule.walkDecls(function (decl) {
    var tokens = decl.value.split(/(,|'[^']*'|"[^"]*")/);
    tokens = tokens.map(function (token, idx) {
      if (idx === 0 || tokens[idx - 1] === ',') {
        var localMatch = /^(\s*):local\s*\((.+?)\)/.exec(token);
        if (localMatch) {
          // 获取作用域名称
          return localMatch[1] + exportScopedName(localMatch[2]) + token.substr(localMatch[0].length);
        } else {
          return token;
        }
      } else {
        return token;
      }
    });
    decl.value = tokens.join('');
  });
});
```

css.walkRules 遍历所有节点信息，无论是 atRule、rule、comment 的父类型，还是 `rule`、 `decl` 的子类型，获取 selector

```js
// 递归遍历节点
function traverseNode(node) {
  switch (node.type) {
    case 'nested-pseudo-class':
      if (node.name === 'local') {
        if (node.nodes.length !== 1) {
          throw new Error('Unexpected comma (",") in :local block');
        }
        return localizeNode(node.nodes[0]);
      }
      /* falls through */
    case 'selectors':
    case 'selector':
      var newNode = Object.create(node);
      newNode.nodes = node.nodes.map(traverseNode);
      return newNode;
  }
  return node;
}
```

walkDecls 遍历每一条规则，生成相应的 Scoped Name

```js
// 生成一个 Scoped Name
function exportScopedName(name) {
  var scopedName = generateScopedName(name, css.source.input.from, css.source.input.css);
  exports[name] = exports[name] || [];
  if (exports[name].indexOf(scopedName) < 0) {
    exports[name].push(scopedName);
  }
  return scopedName;
}
```

关于实现 composes 的组合语法，有点类似，不再赘述

### postcss-modules-values

这个库的主要作用是在模块文件之间传递任意值

它的实现也是只有一个文件，具体[查看这里](https://github.com/css-modules/postcss-modules-values/blob/master/src/index.js)

查看所有的 @value 语句，并将它们视为局部变量或导入的

```js
/* Look at all the @value statements and treat them as locals or as imports */
// 查看所有的 @value 语句，并将它们视为局部变量还是导入的
css.walkAtRules('value', atRule => {
  if (matchImports.exec(atRule.params)) {
    addImport(atRule)
  } else {
    if (atRule.params.indexOf('@value') !== -1) {
      result.warn('Invalid value definition: ' + atRule.params)
    }

    addDefinition(atRule)
  }
})
```

假如是导入的，调用的 addImport 方法

```js
// 添加 import 方法
const addImport = atRule => {
  // 如果有 import 的语法
  let matches = matchImports.exec(atRule.params)
  if (matches) {
    let [/*match*/, aliases, path] = matches
    // We can use constants for path names
    if (definitions[path]) path = definitions[path]
    let imports = aliases.replace(/^\(\s*([\s\S]+)\s*\)$/, '$1').split(/\s*,\s*/).map(alias => {
      let tokens = matchImport.exec(alias)
      if (tokens) {
        let [/*match*/, theirName, myName = theirName] = tokens
        let importedName = createImportedName(myName)
        // 根据 import 的语法，进行定义
        definitions[myName] = importedName
        return { theirName, importedName }
      } else {
        throw new Error(`@import statement "${alias}" is invalid!`)
      }
    })
    importAliases.push({ path, imports })
    atRule.remove()
  }
}
```

否则则直接 addDefinition

```js
// 添加定义
const addDefinition = atRule => {
  let matches
  while (matches = matchValueDefinition.exec(atRule.params)) {
    let [/*match*/, key, value] = matches
    // Add to the definitions, knowing that values can refer to each other
    definitions[key] = replaceAll(definitions, value)
    atRule.remove()
  }
}
```

## 总结

CSS Modules 并不是 CSS 官方的标准，也不是浏览器的特性，而是使用一些构建工具，比如 webpack，对 CSS 类名和选择器限定作用域的一种方式（类似命名空间）。通过 CSS Modules，我们可以实现 CSS 的局部作用域，Class 的组合等功能。最后我们知道 CSS Loader 实际上是通过两个库进行实现的。其中， `postcss-modules-scope` —— 实现CSS Modules 的样式隔离（Scope Local）以及继承（Extend）和 `postcss-modules-values` ——在模块文件之间传递任意值

## 参考

- [开发 postcss 插件](http://echizen.github.io/tech/2017/10-29-develop-postcss-plugin)
- [CSS Modules](https://github.com/css-modules/css-modules)
- [CSS Modules 用法教程](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)

