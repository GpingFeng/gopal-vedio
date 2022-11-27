之前我们探讨了透传属性 $attrs 的用法，今天我们来看下，怎么透彻监听事件。

方法很简单，通过 $listeners 即可实现

> 包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。


<iframe  width="600" height="800"  src="//player.bilibili.com/player.html?aid=817798844&bvid=BV1tG4y147Yy&cid=896762732&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>