## npm script 中使用 cache 导致缓存失效
在一次 `webpack` 优化中，在 `npm script` 中定义了一个命令行参数，在 `webpack` 的配置脚本中使用（主要是用来判别是否开启缓存），类似如下

```json
{
  "scripts": {
	  "dev:cache": "webpack-dev-server --inline --progress --cache=true",
  }
}
```

看似没有没问题，但却命中了 `webpack` 的一个坑，先来看下 `webpack` 的一个配置——[cache](https://webpack.docschina.org/configuration/other-options/#cache)


缓存生成的 webpack 模块和 chunk，来改善构建速度。cache 会在开发 模式被设置成 type: 'memory' 而且在 生产 模式 中被禁用。 cache: true 与 cache: { type: 'memory' } 配置作用一致【也就是默认的话，是会缓存到内存中】。 **传入 `false` 会禁用缓存**:

webpack.config.js

```js
module.exports = {
  //...
  cache: false
};
```

然后看看 `webpack` 的[命令行说明](https://guoyongfeng.github.io/book/07/03-webpack%20%E5%91%BD%E4%BB%A4%E8%A1%8C%E8%AF%B4%E6%98%8E.html)

```
$ webpack --help
webpack 1.13.1
Usage: https://webpack.github.io/docs/cli.html

Options:
  # 省略...
  --cache                                                                                           [default: true]
  --watch, -w
  --watch which closes when stdin ends
  # 省略...
```

其中就包含了 `--cache` 的设置，确实掉进坑里了

反思：
- 命名尽量特殊化