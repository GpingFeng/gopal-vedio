# 深入浅出 CSS Modules

## CSS Modules 是什么？

官方文档的介绍如下：

> A **CSS Modules** is a CSS file in which all class names and animation names are scoped locally by default.

所有的类名和动画名称默认都有各自的作用域的 `CSS` 文件。CSS Modules 并不是 CSS 官方的标准，也不是浏览器的特性，而是使用一些构建工具，比如 webpack，对 CSS 类名和选择器限定作用域的一种方式（类似命名空间）

## CSS Modules 的简单使用

这里我们直接使用 Vue 进行演示

> 很多人都知道 Vue 的 Scoped，这是 CSS Modules 中一个特性的实现（很多时候我们直接使用这个就足够了，但 CSS Modules 的特性远不于此）。实际上 `vue-loader` 提供了与 CSS Modules 的一流集成，可以作为模拟 scoped CSS 的替代方案。



### 配置

新建一个项目，本文的 [Demo]()





```bash
 yarn eject
```



看到 `config/webpack.config.js`，默认情况下，React 脚手架搭建出来的项目，只有 

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







```bash
vue init webpack learn-css-modules
cd learn-css-modules
```

在 `build/utils.js` 中，配置 CSS-loader

```js
const cssLoader = {
  loader: 'css-loader',
  options: {
    sourceMap: options.sourceMap,
    // 开启 CSS Modules
    modules: true,
    // 自定义生成的类名
    localIdentName: '[local]_[hash:base64:8]'
  }
}
```

在 SFC（单文件组件）的 style 标签中加入 `module`

```css
<style module>
h1, h2 {
  font-weight: normal;
}
<style>
```





## 为什么我们要使用 CSS Modules？

局部作用域





## CSS Modules 的实现原理