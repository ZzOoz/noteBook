# Html&Css知识总结



## 重回和重绘的区别和定义：

参考：https://www.jianshu.com/p/e081f9aa03fb

通过 display: none 隐藏一个 DOM 节点-触发重排和重绘
通过 visibility: hidden 隐藏一个 DOM 节点-只触发重绘，因为没有几何变化

## css选择器的优先级

参考：https://blog.csdn.net/it_rod/article/details/80466074

CSS的选择器其实大类的话可以分为三类，即id选择器、class选择器、标签选择器。

用法如下：

- **[id选择器](http://www.w3cschool.cn/cssref/selector-id.html)：** #id名 { 属性名:属性值; }
- **[class选择器](http://www.w3cschool.cn/cssref/selector-class.html) ：**.class名 { 属性名:属性值; }
- **标签选择器：** 标签名 { 属性名:属性值; }

其中，他们之间又可以以不同的方式进行组合，如下：

1. **后代选择器:** 父代名 后代名 { 属性名:属性值; }
2. **子代选择器:**父代名>子代名 { 属性名:属性值; }
3. **群组选择器:** #name1, .name2, #name div { 属性名:属性值; }
4. **伪类选择器:** name:伪类
5. **通用(通配符)选择器:**`* { 属性名: 属性值; }` ……

常用的也就这些。

**优先级：*内联样式 > ID选择器 > 类选择器 = 属性选择器 = 伪类选择器 > 元素选择器 = 关系选择器 = 伪元素选择器 > 通配符选择器**



## Css中的grid布局

## Video标签自定义样式