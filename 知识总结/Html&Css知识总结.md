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



## 文本溢出处理

```js
.single {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    width: 100px;
    height: 100px;
}
.more {
    display: -webkit-box !important;
    overflow: hidden;
    text-overflow: ellipsis;
    word-break: break-all;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
    width: 150px;
}
```



## css属性说明

### 1、**word-break**属性（在合适的点换行）

| normal    | 使用浏览器默认的换行规则。     |
| --------- | ------------------------------ |
| break-all | 允许在单词内换行。             |
| keep-all  | 只能在半角空格或连字符处换行。 |



### 2、box-orient 属性（指定div元素的子元素横向排列）

| horizontal  | 指定子元素在一个水平线上从左至右排列 |
| ----------- | ------------------------------------ |
| vertical    | 从顶部向底部垂直布置子元素           |
| inline-axis | 子元素沿着内联坐标轴（映射到横向）   |
| block-axis  | 子元素沿着块坐标轴（映射到垂直）     |
| inherit     | box-orient 属性的值应该从父元素继承  |



### 3、 line-clamp属性（**限制在一个块元素显示的文本的行数。**）

**感谢参考：**https://www.cnblogs.com/jing-tian/p/11048853.html				

-webkit-line-clamp 是一个 不规范的属性（unsupported WebKit property），它没有出现在 CSS 规范草案中。

为了实现该效果，它需要组合其他外来的WebKit属性。常见结合属性：

- display: -webkit-box; 必须结合的属性 ，将对象作为弹性伸缩盒子模型显示 。
- -webkit-box-orient 必须结合的属性 ，设置或检索伸缩盒对象的子元素的排列方式 。
- text-overflow，可以用来多行文本的情况下，用省略号“...”隐藏超出范围的文本 。

今天接到优化需求，帖子列表里的内容要求缩略至3行，并带‘…’省略号

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <title>cline-clamp</title>
    <style>    
            .box{
                width: 200px;
                height: 300px;
                border:1px solid black;
            }
            p{
                 display: -webkit-box;
                 -webkit-box-orient: vertical;
                  -webkit-line-clamp: 4;            /*设置p元素最大4行，父元素需填写宽度才明显*/
                  text-overflow: ellipsis;
                  overflow: hidden;
                 /* autoprefixer: off */
                 -webkit-box-orient: vertical;
                  /* autoprefixer: on */
                  /*因为代码环境的关系-webkit-box-orient被过滤掉了 autoprefixer 这个关键字可以免除被过滤的动作*/
　　　　　　　　　　word-wrap:break-word;
　　　　　　　　　　word-break:break-all;
} </style> 
</head> 
<body> 
<div class="box"> 
    <p> static：对象遵循常规流。top，right，bottom，left等属性不会被应用。 relative： 对象遵循常规流，并且参照自身在常规流中的位置通过top，right，bottom，left属性进行偏移时不影响常规流中的任何元素。 absolute：对象脱离常规流，使用top，right，bottom，left等属性进行绝对定位，
    </p> 
</div> 
</body> 
</html>
```



### 4.background属性