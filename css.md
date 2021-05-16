## CSS盒模型

- 基本概念

  margin,border,padding,content

- 标准模型和IE模型

  IE模型：width=content+padding

  标准模型：width=content

- 如何设置

  box-size: 

  - content-box: 标准模型
  - border-box: IE模型



## [css选择器](https://segmentfault.com/a/1190000013424772)

- `简单选择器（Simple selectors）`：通过元素类型、class 或 id 匹配一个或多个元素。
- `属性选择器（Attribute selectors）`：通过 属性 / 属性值 匹配一个或多个元素。
- `伪类（Pseudo-classes）`：匹配处于确定状态的一个或多个元素，比如被鼠标指针悬停的元素，或当前被选中或未选中的复选框，或元素是 DOM 树中一父节点的第一个子节点。
- `伪元素（Pseudo-elements）`:匹配处于相关的确定位置的一个或多个元素，例如每个段落的第一个字，或者某个元素之前生成的内容。
- `组合器（Combinators）`：这里不仅仅是选择器本身，还有以有效的方式组合两个或更多的选择器用于非常特定的选择的方法。例如，你可以只选择 divs 的直系子节点的段落，或者直接跟在 headings 后面的段落。
- `多元素选择器（Multiple selectors）`：这些也不是单独的选择器；这个思路是将以逗号分隔开的多个选择器放在一个 CSS 规则下面， 以将一组声明应用于由这些选择器选择的所有元素。



## [BFC模型](https://zhuanlan.zhihu.com/p/25321647)

- BFC 即 Block Formatting Contexts (块级格式化上下文)

  **具有 BFC 特性的元素可以看作是隔离了的独立容器，容器里面的元素不会在布局上影响到外面的元素，并且 BFC 具有普通容器所没有的一些特性。**

  通俗一点来讲，可以把 BFC 理解为一个封闭的大箱子，箱子内部的元素无论如何翻江倒海，都不会影响到外部。

- 怎么触发BFC

  只要元素满足下面任一条件即可触发 BFC 特性：

  - body 根元素

  - 浮动元素：float 除 none 以外的值

  - 绝对定位元素：position (absolute、fixed)

  - display 为 inline-block、table-cells、flex

  - overflow 除了 visible 以外的值 (hidden、auto、scroll)

    

- BFC特性及其应用

  **1. 同一个 BFC 下外边距会发生折叠**

  **2. BFC 可以包含浮动的元素（清除浮动）**

  **3. BFC 可以阻止元素被浮动元素覆盖**



## [Postion(定位)](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS_layout/Positioning)

- position: static/relative/absolute/fixed/sticky

  - static: 默认， 处于正常文档流

  - relative: 相对定位， 仍然处于正常文档流，但是可以通过top/left/bottom/right修改其位置

  - absolute: 绝对定位，脱离文档流，定位依据是最后一个父级及以上position不为static的元素

  - Fixed: 固定定位，脱离文档流，定位依据是视口

  - sticky: 基本上是相对位置和固定位置的混合体，它允许被定位的元素表现得像相对定位一样，直到它滚动到某个阈值点（例如，从视口顶部起10像素）为止，此后它就变得固定了

    示例代码:

    ```html
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>菜鸟教程(runoob.com)</title>
    </head>
    <style>
    dt {
      background-color: black;
      color: white;
      padding: 10px;
      position: sticky;
      top: 0;
      left: 0;
      margin: 1em 0;
    }
    </style>
    <body>
    
    <h1>Sticky positioning</h1>
    
    <dl>
        <dt>A</dt>
        <dd>Apple</dd>
        <dd>Ant</dd>
        <dd>Altimeter</dd>
        <dd>Airplane</dd>
        <dt>B</dt>
        <dd>Bird</dd>
        <dd>Buzzard</dd>
        <dd>Bee</dd>
        <dd>Banana</dd>
        <dd>Beanstalk</dd>
        <dt>C</dt>
        <dd>Calculator</dd>
        <dd>Cane</dd>
        <dd>Camera</dd>
        <dd>Camel</dd>
        <dt>D</dt>
        <dd>Duck</dd>
        <dd>Dime</dd>
        <dd>Dipstick</dd>
        <dd>Drone</dd>
        <dt>E</dt>
        <dd>Egg</dd>
        <dd>Elephant</dd>
        <dd>Egret</dd>
    </dl>
    </body>
    </html>
    ```

    

## [flex布局](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox)



## [css优先级](https://zhuanlan.zhihu.com/p/41604775)

1. 常用选择器权重优先级：***!important > id > class > tag\***
2. !important可以提升样式优先级，但不建议使用。如果!important被用于一个简写的样式属性，那么这条简写的样式属性所代表的子属性都会被应用上!important。 例如：*background: blue !important;*
3. 如果两条样式都使用!important，则权重值高的优先级更高
4. 在css样式表中，同一个CSS样式你写了两次，后面的会覆盖前面的
5. 样式指向同一元素，权重规则生效，权重大的被应用
6. 样式指向同一元素，权重规则生效，权重相同时，就近原则生效，后面定义的被应用
7. 样式不指向同一元素时，权重规则失效，就近原则生效，离目标元素最近的样式被应用



## [深入理解圣杯布局和双飞翼布局](https://juejin.cn/post/6844903817104850952)



## [css3新特性](https://segmentfault.com/a/1190000010780991)



## [css样式隔离](https://juejin.cn/post/6844904034281734151#heading-9)



## [css性能优化](https://blog.csdn.net/weixin_43883485/article/details/103504171)

1. 合并css文件，如果页面加载10个css文件,每个文件1k，那么也要比只加载一个100k的css文件慢。
2. 减少css嵌套，最好不要嵌套三层以上。
3. 不要在ID选择器前面进行嵌套，ID本来就是唯一的而且权限值大，嵌套完全是浪费性能。
4. 建立公共样式类，把相同样式提取出来作为公共类使用。
5. 减少通配符*或者类似[hidden="true"]这类选择器的使用，挨个查找所有...这性能能好吗？
6. 巧妙运用css的继承机制，如果父节点定义了，子节点就无需定义。
7. 拆分出公共css文件，对于比较大的项目可以将大部分页面的公共结构样式提取出来放到单独css文件里，这样一次下载 后就放到缓存里，当然这种做法会增加请求，具体做法应以实际情况而定。
8. 不用css表达式，表达式只是让你的代码显得更加酷炫，但是对性能的浪费可能是超乎你想象的。
9. 少用css rest，可能会觉得重置样式是规范，但是其实其中有很多操作是不必要不友好的，有需求有兴趣，可以选择normolize.css。
10. cssSprite，合成所有icon图片，用宽高加上background-position的背景图方式显现icon图，这样很实用，减少了http请求。
11. 善后工作，css压缩(在线压缩工具 YUI Compressor)
12. GZIP压缩，是一种流行的文件压缩算法。



## [CSS中的层叠上下文](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)



## [div居中](https://juejin.cn/post/6844903821529841671)



## [css 浮动](https://segmentfault.com/a/1190000012739764)