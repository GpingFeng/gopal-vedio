### [选择某类的最后一个元素](https://juejin.cn/post/6844904072206614535) 

- :first-child / :last-child
  - :first-child：选择属于其父元素的首个子元素的每个子元素
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
