# Vue-i18n 变量使用方式以及采坑总结

## 前言

笔者目前在 `Shopee` 工作，我们公司主要业务是跨境电商，业务涉及到多个国家，所以我们各个系统都会涉及到国际化翻译。我们 `Vue` 项目技术上采用了 `Vue-i18n` 这个库。

今天就聊聊这个库的一个功能，在国际化时候使用变量。在翻译中使用变量是一个非常常见的场景，最简单的例子，比如以下的文案要国际化

```
I am Gopal.I am from China
```

但其中 **Gopal** 和 **China** 是需要变量传入的，这个时候我们怎么办呢？

## 简单的传参

首先我们定义要翻译的字符如下

```
"introTips": "I am {name}.I am from {region}"
```

然后在模板中使用

```js
$t('introTips', { name: 'Gopal一号', region: 'China' })
```

就可以渲染出 **I am Gopal 一号.I am from China**



## 需要给变量加个颜色

假如说我们 **Gopal** 不仅仅是一个文案，还需要变成红色？我们该怎么办？我们可以直接使用 v-html 渲染 html。这个时候我们就要修改翻译的字符如下

```
introTipsTwo: "I am <span class='name'>{name}</span>.I am from {region}"
```

使用 v-html 直接渲染

```html
<div
  class="text"
  v-html="$t('introTipsTwo', { name: 'Gopal一号', region: 'China' })"
></div>
```

渲染结果如下

```html
<div data-v-763db97b="" class="text">
  I am <span class="name">Gopal一号</span>.I am from China
</div>
```



![image-20210430112350433](https://tva1.sinaimg.cn/large/008i3skNgy1gq1lvraaklj31ew0b80v7.jpg)

**注意：这个方法很可能引发 XSS 攻击，所以不推荐使用该方法**



## 使用 place 属性

首先翻译的文案先改回最开始变量的版本

```
introTips: "I am {name}.I am from {region}"
```



直接使用 i18n 组件以及 place 属性，其中 path 指的是上面的 key，place 指定变量

```html
<i18n path="introTips" tag="div">
  <span class="name" place="name">Gopal 二号</span>
  <span place="region">China</span>
</i18n>
```



渲染结果如下：

```html
<div data-v-763db97b="">
  I am <span data-v-763db97b="" place="name" class="name">Gopal 二号</span>.I am from <span data-v-763db97b="" place="region">China</span>
</div>
```



![image-20210430113555745](https://tva1.sinaimg.cn/large/008i3skNgy1gq1m89pql9j31jg0akq5o.jpg)

我们已经实现了我们的方案，但**这个方法会在下个版本中被淘汰**，仔细看，这不是 Vue 中的插槽么？没错，官方推荐的最终的解决方案是 Slot，用法跟上面的非常相似

## 最终的方案——Slot

直接上代码：

```html
<i18n path="introTips" tag="div">
  <template v-slot:name>
    <span class="name">Gopal 三号</span>
  </template>
  <template v-slot:region>
    <span>China</span>
  </template>
</i18n>
```

渲染的结果跟上面的类似

```html
<div data-v-763db97b="">
  I am <span data-v-763db97b="" class="name">Gopal 三号</span>.I am from <span data-v-763db97b="">China</span></div>
```



## 问题

在项目中使用的时候，却报错了...

我的办法跟上面的可谓是一模一样...

### 报错

```html
<i18n path="introTips" tag="div">
  <template v-slot:name>
    <span class="name">Gopal 三号</span>
  </template>
  <template v-slot:region>
    <span>China</span>
  </template>
</i18n>
```

```js
vue.esm.js:628 [Vue warn]: Error in nextTick: "TypeError: Cannot create property 'isRootInsert' on string 'You submit '"
```



![](https://tva1.sinaimg.cn/large/008i3skNgy1gq1mhnu6apj30s20dawij.jpg)



我感觉我又要掉头发了...。

### 断点查看

`Google` 了一下这个报错以及看了一下报错的堆栈信息，看起来像是 vNode 渲染的问题，然后我在报错的地方打了一个断点，可以看到下面 children 中数组的各项并不都是 vnode，第一项就是字符串，这应该就是导致报错的罪魁祸首



![](https://tva1.sinaimg.cn/large/008i3skNgy1gq5h43rypqj30rx0laadb.jpg)



### 阅读源码

我看了一下 node_modules 中 `vue-i18n` 的源码，这里注意我目前使用的版本是 **8.15.0**

发现在 i18n 这个函数式组件的源码中有两句非常奇怪的代码，这个函数式组件源码见[链接](https://github.com/kazupon/vue-i18n/blob/v8.15.0/src/components/interpolation.js#L42)

```js
export default {
  name: 'i18n',
  functional: true,
  props: {
    // 留意这里
    tag: {
      type: String
    },
		// ...
  },
  render (h: Function, { data, parent, props, slots }: Object) {
    const { $i18n } = parent
		// ...
    const { path, locale, places } = props
    const params = slots()
    const children = $i18n.i(
      path,
      locale,
      onlyHasDefaultPlace(params) || places
        ? useLegacyPlaces(params.default, places)
        : params
    )

    const tag = props.tag || 'span'
    return tag ? h(tag, data, children) : children
  }
}
```

留意最后两行代码

```js
const tag = props.tag || 'span'
return tag ? h(tag, data, children) : children
```

 啊，这。。。`children` 是不是永远没有机会执行了？

我尝试直接 return 回 `children`。额成功了...

### 尝试升级版本

我想了想，升级到较新的版本试下？

果然，这个组件修改了...，简化了一下代码，源码可见[链接](https://github.com/kazupon/vue-i18n/blob/v8.22.0/src/components/interpolation.js)

```js
export default {
  name: 'i18n',
  functional: true,
  props: {
    // 留意这里
    tag: {
      type: [String, Boolean, Object],
      default: 'span'
    }
    // ...
  },
  render (h: Function, { data, parent, props, slots }: Object) {
    // ....
		// 留意这里
    const tag = (!!props.tag && props.tag !== true) || props.tag === false ? props.tag : 'span'
    return tag ? h(tag, data, children) : children
  }
}
```

这里的 tag 可以传 String，Boolean 和 Object

看了一下官方文档

> You can choose the root container's node type by specifying a `tag` prop. If omitted, it defaults to `'span'`. You can also set it to the boolean value `false` to insert the child nodes directly without creating a root element.

简单来说 **tag 可以传 false，这样就不需要在翻译的外层再多加一个节点了**

我再去找了一下修改这个 [PR](https://github.com/kazupon/vue-i18n/pull/878) 以及对应的 [Issue](https://github.com/kazupon/vue-i18n/issues/876)

### 解决问题

虽然看起来为了解决不需要配置根节点的问题的，但我感觉也可以解决我的问题，以下升级版本后，我修改了一下我的代码

```html
<div class="test-container">
  <i18n path="introTips" :tag="false">
    <template v-slot:name>
      <span class="name">Gopal 三号</span>
    </template>
    <template v-slot:region>
      <span>China</span>
    </template>
  </i18n>
</div>
```

这回真的成功了，非常开心，最终它的渲染如下

```html
<div data-v-0b89d11d="" class="test-container">
  <!-- 这里外层没有节点了 -->
  I am <span data-v-0b89d11d="" class="name">Gopal 三号</span>.I am from <span data-v-0b89d11d="">China</span></div>
```

可以看到这个时候渲染出来就没有最外层的 tag 了

## 总结

本文介绍了 vue-i18n 变量的使用方法，几种方法都较为简单易懂。主要是记录了自己踩过的一个坑，也算是学到了一些经验

- 当没有思路的时候，可以看看源码。可以直接 node_modules 中查看，甚至在 dist  文件中找到相对应的代码，断点调试
- 源码调试不仅仅对定位问题有所帮助，也可以扩宽视野。比如上面提到的 i18n 函数式组件的实现。我当时看的时候，假如让我来写，我可以写出来么？
- 留意官方的 ISSUE，不过这次前期我都找过，只是都不是我这个错误...
- 官方的文档英文比较全，中文版和英文版差异较大（应该是翻译滞后了）



## 最后

之前提到了，笔者在 Shopee 工作，有需要换工作的同学可以找我内推哈~

五月份专场非常多机会，具体可以查看 [链接](https://app.mokahr.com/apply/shopee/2963#/)。一线大厂薪资，少加班（公司提倡高效工作），团队氛围好，晋升机会多，感兴趣的同学请抓紧时间，发送简历到我邮箱 15521091584@163.com



![](https://tva1.sinaimg.cn/large/008i3skNgy1gq5hf56hscj319i0u00yq.jpg)