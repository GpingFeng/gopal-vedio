
### [伪类不生效的原因](https://www.html.cn/qa/css3/15060.html)

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

### content 属性的探索

MDN 中的 [content 属性](https://developer.mozilla.org/zh-CN/docs/Web/CSS/content) 的解释

以下为 content 属性的一些作用，重点会说下 counter()

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

