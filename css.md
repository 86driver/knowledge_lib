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

