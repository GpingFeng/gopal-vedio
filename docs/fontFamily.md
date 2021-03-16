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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5ea418dc4364ea4b616b678611f88c7~tplv-k3u1fbpfcp-zoom-1.image ":size=500")

但是，我们给子元素的 `font-family` 随便设置一个值 test，这实际上是一个无效的字体

```css
.child {
  font-family: test;
}
```

奇怪的事情来了，浏览器竟然识别不出来这是一个无效的值，计算值的结果还是 test（这里猜测因为 `font-family` 可以通过 `@font-face` 自定义设置，所以浏览器无法知道它是无效的），但实际上效果已经直接降级到浏览器的默认值了，而不是父级元素设置的值，这跟我们认知的还不太一样....

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9c600a10fa740cab46762875ae98e9d~tplv-k3u1fbpfcp-zoom-1.image ":size=500")

假如我们设置子元素的样式如下，即在 test 之后再设置一个字体 `Gill Sans`

```css
.child {
  font-family: test, "Gill Sans";
}
```

这个时候，可以看到降级到 `Gill Sans` 了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/507ddc6d863c4a51ae767ab29f46a103~tplv-k3u1fbpfcp-zoom-1.image ":size=500")

[demo 地址](https://codepen.io/gpingfeng/pen/KKgoVoJ)

**小总结**:

假如你设置了 `font-family`，而且不知道自己设置的字体能不能在所有的浏览器上都能生效的时候，推荐总写一个兜底的字体。否则假如该字体不生效的话，该元素并不会自动继承你在 `body` 或者 `html` 元素等父级元素中设置的兜底值
