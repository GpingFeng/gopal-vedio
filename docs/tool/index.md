## [network 中的 filter](https://juejin.cn/post/6850418118083215368)

- 一般过滤很简单，比如你想查找所有.css 文件，那么就输入： .css
- 高级过滤
  - `-`（中划线）类似：`-method:OPTIONS`
  - 正则

## [XMLHttpRequest.responseType](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/responseType)

XMLHttpRequest.responseType 属性是一个枚举类型的属性，返回响应数据的类型。它允许我们手动的设置返回数据的类型。如果我们将它设置为一个空字符串，它将使用默认的 "text" 类型。

在工作环境 (Work Environment) 中将 responseType 的值设置为 "document" 通常会被忽略。当将 responseType 设置为一个特定的类型时，你需要确保服务器所返回的类型和你所设置的返回值类型是兼容的。那么如果两者类型不兼容呢？恭喜你，你会发现服务器返回的数据变成了 `null`，即使服务器返回了数据。还有一个要注意的是，给一个同步请求设置 responseType 会抛出一个 InvalidAccessError 的异常。

> 比如在项目中遇到的场景就是需要设置 `responseType` 为 `blob`，代表 response 是一个包含二进制数据的 Blob 对象 。

## Git 无法检测到文件名大小写的更改

解决方法一：通过 `git config`

```
$ git config core.ignorecase false
```

解决方法二：`git mv`

```
git mv myfolder tmp
git mv tmp MyFolder
```