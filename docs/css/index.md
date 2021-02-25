## font-family 继承失效了？

`CSS` 属性 `font-family` 允许您通过给定一个有先后顺序的，由字体名或者字体族名组成的列表来为选定的元素设置字体。

属性 `font-family` 列举一个或多个由逗号隔开的字体族，语法如下：

```css
/* 一个字体族名和一个通用字体族名 */
font-family: "Gill Sans Extrabold", sans-serif;
font-family: "Goudy Bookletter 1911", sans-serif;

/* 仅有一个通用字体族名 */
font-family: serif;
font-family: sans-serif;
font-family: monospace;
font-family: cursive;
font-family: fantasy;
font-family: system-ui;
font-family: emoji;
font-family: math;
font-family: fangsong;

/* 全局值 */
font-family: inherit;
font-family: initial;
font-family: unset;
```

`font-family` 是一个继承属性，比如，我给上面例子 `.parent` 添加 `font-family: serif;`，子元素也会继承它的属性值

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5ea418dc4364ea4b616b678611f88c7~tplv-k3u1fbpfcp-zoom-1.image)

但是，我们给子元素的 `font-family` 随便设置一个值 test，这实际上是一个无效的字体

```css
.child {
  font-family: test;
}
```

奇怪的事情来了，浏览器竟然识别不出来这是一个无效的值，计算值的结果还是 test（这里猜测因为 `font-family` 可以通过 `@font-face` 自定义设置，所以浏览器无法知道它是无效的），但实际上效果已经直接降级到浏览器的默认值了，而不是父级元素设置的值，这跟我们认知的还不太一样....

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9c600a10fa740cab46762875ae98e9d~tplv-k3u1fbpfcp-zoom-1.image)

假如我们设置子元素的样式如下，即在 test 之后再设置一个字体 `Gill Sans`

```css
.child {
  font-family: test, "Gill Sans";
}
```

这个时候，可以看到降级到 `Gill Sans` 了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/507ddc6d863c4a51ae767ab29f46a103~tplv-k3u1fbpfcp-zoom-1.image)

[demo 地址](https://codepen.io/gpingfeng/pen/KKgoVoJ)

**小总结**:

假如你设置了 `font-family`，而且不知道自己设置的字体能不能在所有的浏览器上都能生效的时候，推荐总写一个兜底的字体。否则假如该字体不生效的话，该元素并不会自动继承你在 `body` 或者 `html` 元素等父级元素中设置的兜底值

## [伪类不生效的原因](https://www.html.cn/qa/css3/15060.html)

`css before` 伪类元素要显示，需要设置 content 属性，而且要具有一定的宽高，可以设置 display 为 inline-block 和 block，让元素的宽高生效，类似如下

```css
.before:before{
  content:"";
  display: inline-block;
  width: 10px;
  height: 10px;
  background: rgba(255,0,0,.3);
}
```

[关于 content  属性](https://developer.mozilla.org/zh-CN/docs/Web/CSS/content)

- 填充字符串
-  `url()`、`URI()`。 URI 值会指定一个外部资源（比如图片）。如果该资源或图片不能显示，它就会被忽略或显示一些占位（比如无图片标志）
-  attr(X)
-  `counter()`, 调用计数器，可以不使用列表元素实现序号问题。这个可以看这篇文章 [使用 CSS 计数器花式玩转列表编号](https://segmentfault.com/a/1190000038292189)，我总结了一下，有以下几个步骤
   -  首先，使用 `counter-reset` 属性设置一个计数器
   -  使用 `counter-increment` 属性增加计数器的值
   -  最后，在内容属性和 `counter()` 函数中使用 `:before` 伪元素来显示数字

```html
<div class="list">
  <div>My first item</div>
  <div>My second item</div>
  <div>My third item</div>
</div>
```
```css
<style>
.list {
  counter-reset: list-number;
}
/** 注意，我们可以在:before伪元素中使用counter-increment **/
.list div:before {
  counter-increment: list-number;
  content: counter(list-number);
}
</style>
```

## [选择某类的最后一个元素](https://juejin.cn/post/6844904072206614535) 

- :first-child / :last-child
  - :first-child：选择属于其父元素的首个子元素的每个子元
  - :last-child：选择属于其父元素的最后一个子元素的每个子元素

> 总结：
> - 想要通过这组选择器选择某一类型首个或最后一个元素需要满足
> - 该元素属于这个类名。
> - 该元素是其父元素下的首个或最后一个子元素

- :first-of-type / :last-of-type
  - :first-of-type：选择属于其父元素的首个指定类型子元素的每个子元素
  - :last-of-type：选择属于其父元素的最后一个指定类型子元素的每个子元素

> - 找到指定类型的元素的标签类型
> - 寻找其父元素下符合该标签类型的元素
> - 当且仅当元素符合选择器中指定类型，且是父元素下同类标签中首个或最后一个子元素时，方可被选中

- :nth-child(n) / :nth-last-child(n)
  - :nth-child (n)：选择属于其父元素的第 n 个子元素，不论元素的类型
  - :nth-last-child (n)：选择属于其父元素的倒数第 n 

> - 总结
> - 当父元素下只有指定的类型：使用:first-child / :last-child
> - 当父元素下除了指定类型外，还有其他标签元素：使用:first-of-type / :last-of-type
> - 当父元素下除了指定类型（某一类名）外，还有其他同类标签元素，且已知其他标签个数：使用:nth-child (n) / :nth-last-child (n) 或:nth-of-type (n) / :nth-last-of-type (n)
