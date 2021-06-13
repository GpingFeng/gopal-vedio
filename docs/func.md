## 函数式组件报错

回顾一下报错



![image-20210613173419032](https://tva1.sinaimg.cn/large/008i3skNgy1grgrurv3vcj31ki0ba430.jpg)



关键是调用 `h(tag, data, children)` 返回有什么问题？

![image-20210613173508438](https://tva1.sinaimg.cn/large/008i3skNgy1grgrvkhubjj312e0aatbp.jpg)



调用 createElement 方法



调用 _createElement 方法



![](https://tva1.sinaimg.cn/large/008i3skNgy1grgrwtrze0j30ts17an7e.jpg)



关注这两个参数

![image-20210613173808801](https://tva1.sinaimg.cn/large/008i3skNgy1grgryq5of6j31280iwdnd.jpg)



因为这两个参数值不对，导致最后的值 children 不是 vnode

![image-20210613173928877](https://tva1.sinaimg.cn/large/008i3skNgy1grgs03r8gcj30ze0j443m.jpg)



![image-20210613174035733](https://tva1.sinaimg.cn/large/008i3skNgy1grgs1akwguj31qc0jegw0.jpg)

找到 设置的点

```js
var isCompiled = isTrue(options._compiled); // true
var needNormalization = !isCompiled; // false
```

假如我手动修改 needNormalization 为 true



但是这里这个 _compiled 没找到是在哪设置的...【查找了一下资料，说是在 vue-loader 中设置的】

https://github.com/bootstrap-vue/bootstrap-vue/issues/6223



## 结论

- VNode 的 Children 最后肯定是 Vnode
- render function 之所以可以写 String，是因为有像类似 `normalizeChildren` 和 `simpleNormalizeChildren`函数将其转化
- 尽量不要在函数式组件 render 返回中使用 string
- 函数式组件的转换与否，靠 `options._compiled` ，这个目前的定位是在 Vue-Loader 中实现的【源码中修改了，无效，需打一个问题】



