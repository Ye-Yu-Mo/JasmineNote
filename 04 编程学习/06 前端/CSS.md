---
title: CSS基础
date: 2024-02-06 15:41:09
tags: [Web,HTML,CSS]
categories: [Web]
---

# CSS基础

层叠样式表 (Cascading Style Sheets，缩写为 CSS）是一种 **样式表** 语言，用来**描述 HTML 文档的呈现**（**美化内容**）。

一般在**title 标签下方添加 style 双标签，style 标签里面书写 CSS 代码**。

```html
<title>CSS 初体验</title>
<style>
  /* 选择器 { } */
  p {
    /* CSS 属性 */
    color: red;
  }
</style>

<p>CSS</p>
```

> 属性名和属性值成对出现 → 键值对。 

## CSS引入方式

* **内部**样式表：学习使用
  * CSS 代码写在 style 标签里面
* **外部**样式表：开发使用
  * CSS 代码写在单独的 CSS 文件中（**.css**）
  * 在 HTML 使用 link 标签引入

```html
<link rel="stylesheet" href="./my.css">
```

* **行内**样式：配合 JavaScript 使用
  * CSS 写在标签的 style 属性值里

```html
<div style="color: red; font-size:20px;">这是 div 标签</div>
```

## 选择器

作用是**查找标签**，设置样式。 

### 标签选择器

标签选择器：使用**标签名**作为选择器 → 选中**同名标签设置相同的样式**。

例如：p, h1, div, a, img......

```html
<style>
  p {
    color: red;
  }
</style>

```

> 标签选择器**无法差异化**同名标签的显示效果。

### 类选择器

作用是查找标签，**差异化**设置标签的显示效果。

步骤：

* 定义类选择器 → **.类名**
* 使用类选择器 → 标签添加 **class="类名"**

```html
<style>
  /* 定义类选择器 */
  .red {
    color: red;
  }
</style>

<!-- 使用类选择器 -->
<div class="red">这是 div 标签</div>
<div class="red size">div 标签</div>
```

### id选择器

作用类似于类选择器，但是id选择器一般**配合 JavaScript** 使用，很少用来设置 CSS 样式

步骤：

* 定义 id 选择器 → #id名
* 使用 id 选择器 → 标签添加 id= "id名"

```html
<style>
  /* 定义 id 选择器 */
  #red {
    color: red;
  }
</style>

<!-- 使用 id 选择器 -->
<div id="red">这是 div 标签</div>
```

需要注意的是同一个 id 选择器在一个页面只能使用一次。

### 通配符选择器

作用：查找页面**所有**标签，设置相同样式。

通配符选择器： ***，不需要调用**，浏览器自动查找页面所有标签，设置相同的样式

```css
* {
  color: red;
}
```

> 通配符选择器可以用于**清除标签的默认样式**，例如标签默认的外边距、内边距。

![1680317584651.png](https://s2.loli.net/2024/02/06/eTfvlKu1psZoY2S.png)

## 盒子尺寸和背景色

| 属性             | 作用   |
| ---------------- | ------ |
| width            | 宽度   |
| height           | 高度   |
| background-color | 背景色 |



## 文字控制属性

### 字体大小

* 属性名：**font-size**
* 属性值：文字尺寸，PC 端网页最常用的单位 **px**

```css
p {
  font-size: 30px;
}
```

> 谷歌浏览器默认字号是16px。

### 字体样式（是否倾斜） 

作用：清除文字默认的倾斜效果

属性名：**font-style**

属性值

* 正常（不倾斜）：**normal** 
* 倾斜：**italic**

### 行高

作用：设置多行文本的间距

属性名：line-height

属性值

* 数字 + px
* 数字（当前标签font-size属性值的倍数）

```css
line-height: 30px;
/* 当前标签字体大小为16px */
line-height: 2;
```

![1680317770048.png](https://s2.loli.net/2024/02/06/KdGm2yzwDjMIa1b.png)

> 行高的测量方法：从一行文字的最顶端（最底端）量到下一行文字的最顶端（最底端）。 

#### 单行文字垂直居中

垂直居中技巧：**行高属性值等于盒子高度属性值**

注意：该技巧适用于单行文字垂直居中效果

```css
div {
  height: 100px;
  background-color: skyblue;
  /* 注意：只能是单行文字垂直居中 */
  line-height: 100px;
}
```

### 字体族

属性名：**font-family**

属性值：字体名

```css
font-family: 楷体;
```

> 拓展（了解）：font-family属性值可以书写多个字体名，各个字体名用逗号隔开，执行顺序是从左向右依次查找
>
> *  font-family 属性最后设置一个字体族名，网页开发建议使用无衬线字体

![1680318278244.png](https://s2.loli.net/2024/02/06/FShMcrAB8dovC25.png)

```css
font-family: Microsoft YaHei, Heiti SC, tahoma, arial, Hiragino Sans GB, "\5B8B\4F53", sans-serif;
```

### font复合属性

使用场景：设置网页文字公共样式 

![1680318326214.png](https://s2.loli.net/2024/02/06/Ufd5GCDujHaqOIy.png)

复合属性：属性的简写方式，**一个属性对应多个值的写法**，各个属性值之间用**空格**隔开。

**font: 是否倾斜  是否加粗  字号/行高 字体（必须按顺序书写）**

```css
div {
  font: italic 700 30px/2 楷体;
}
```

> 字号和字体值必须书写，否则 font 属性不生效 。

### 文本缩进 

属性名：**text-indent**

属性值：

* 数字 + px
* **数字 + em**（推荐：**1em = 当前标签的字号大小**）

```css
p {
  text-indent: 2em;
}
```

### 文本对齐方式 

作用：控制内容水平对齐方式

属性名：**text-align**

| 属性   | 效果           |
| ------ | -------------- |
| left   | 左对齐（默认） |
| center | 居中           |
| right  | 右对齐         |



```css
text-align: center;
```

> text-align本质是控制内容的对齐方式，属性要设置给内容的父级。 

```html
<style>
  div {
    text-align: center;
  }
</style>

<div>
  <img src="./images/1.jpg" alt="">
</div>
```

### 文本修饰线 

属性名： **text-decoration** 

| 属性         | 效果   |
| ------------ | ------ |
| none         | 无     |
| underline    | 下划线 |
| line-through | 删除线 |
| overline     | 上划线 |



### color 文字颜色

| 颜色表示     | 属性值        | 说明                           | 场景         |
| ------------ | ------------- | ------------------------------ | ------------ |
| 颜色关键字   | 颜色单词      | red\green\blue                 | 学习         |
| RGB表示      | rgb(r,g,b)    | rgb表示红绿蓝三原色，取值0~255 | 了解         |
| rgba表示     | rgba(r,g,b,a) | a表示透明度，取值0~1           | 开发，透明色 |
| 十六进制表示 | #RRGGBB       | #000000，#00ffcc00             | 开发         |



> 提示：只要属性值为颜色，都可以使用上述四种颜色表示方式，例如：背景色。 

## 06-调试工具

作用：检查、调试代码；帮助程序员发现代码问题、解决问题

1. 打开调试工具

* 浏览器窗口内任意位置 / 选中标签 → 鼠标右键 → 检查
* F12

2. 使用调试工具

![1680318624523.png](https://s2.loli.net/2024/02/06/dNAuSw348EH6zfo.png)

# CSS进阶

## 复合选择器

复和选择器是由两个或多个基础选择器，通过不同的方式组合而成，可以更准确、更高效的选择目标元素（标签）。

### 后代选择器

后代选择器用于**选中某元素的后代元素**。

选择器写法：父选择器  子选择器 { CSS 属性}，父子选择器之间用**空格**隔开。

```html
<style>
  div span {
    color: red;
  }
</style>

<span> span 标签</span>
<div>
  <span>这是 div 的儿子 span</span >
</div>
```

![屏幕截图 2024-02-15 124602.png](https://s2.loli.net/2024/02/15/snACzp7OWYNcBRJ.png)

### 子代选择器

子代选择器用于选中某元素的子代元素（**最近的子级**）。

选择器写法：父选择器 > 子选择器 { CSS 属性}，父子选择器之间用 **>** 隔开。

```html
<style>
  div > span {
    color: red;
  }
</style>

<div>
  <span>这是 div 里面的 span</span>
  <p>
    <span>这是 div 里面的 p 里面的 span</span>
  </p>
</div>
```

![image-20240215124724099](https://s2.loli.net/2024/02/15/AOZ5rK8h4JzQTDq.png)

### 并集选择器

并集选择器用于选中**多组标签**设置**相同**的样式。

选择器写法：选择器1, 选择器2, …, 选择器N { CSS 属性}，选择器之间用 **,** 隔开。

```html
<style>
  div,
  p,
  span {
    color: red;
  }
</style>

<div> div 标签</div>
<p>p 标签</p>
<span>span 标签</span>
```

![屏幕截图 2024-02-15 125006.png](https://s2.loli.net/2024/02/15/7Atq4CFvxHEl5zS.png)

### 交集选择器 

交集选择器：选中**同时满足多个条件**的元素。

选择器写法：选择器1选择器2 { CSS 属性}，选择器之间连写，没有任何符号。 

```html
<style>
  p.box {
  color: red;
}
</style>

<p class="box">p 标签，使用了类选择器 box</p>
<p>p 标签</p>
<div class="box">div 标签，使用了类选择器 box</div>
```

> 注意：如果交集选择器中有标签选择器，标签选择器必须书写在最前面。 

![屏幕截图 2024-02-15 125317.png](https://s2.loli.net/2024/02/15/rRoOW8HhenYyzg2.png)

### 伪类选择器 

伪类选择器是伪类表示元素**状态**，选中元素的某个状态设置样式。

鼠标悬停状态：**选择器:hover { CSS 属性 }**

```html
<style>
  a:hover {
    color: red;
  }
  .box:hover {
    color: green;
  }
</style>

<a href="#">a 标签</a>
<div class="box">div 标签</div>
```

![屏幕截图 2024-02-15 125437.png](https://s2.loli.net/2024/02/15/zCqERG72wPufTSU.png)

#### 超链接伪类

| 选择器   | 作用   |
| -------- | ------ |
| :link    | 访问前 |
| :visited | 访问后 |
| :hover   | 悬停   |
| :active  | 激活   |



> 如果要给超链接设置以上四个状态，需要按 LVHA 的顺序书写。 工作中，一个 a 标签选择器设置超链接的样式， hover状态特殊设置 
>

```css
a {
  color: red;
}

a:hover {
  color: green;
}
```

## CSS特性

CSS特性：化简代码 / 定位问题，并解决问题

* 继承性
* 层叠性
* 优先级

### 继承性

继承性：子级默认继承父级的**文字控制属性**。 

![1680319376438.png](https://s2.loli.net/2024/02/15/vJoj1kizGHCyNVY.png)

> 注意：如果标签有默认文字样式会继承失败。 例如：a 标签的颜色、标题的字体大小。

### 层叠性

特点：

* 相同的属性会覆盖：**后面的 CSS 属性覆盖前面的 CSS 属性**
* 不同的属性会叠加：**不同的 CSS 属性都生效**

```html
<style>
  div {
    color: red;
    font-weight: 700;
  }
  div {
    color: green;
    font-size: 30px;
  }
</style>

<div>div 标签</div>
```

> 注意：选择器类型相同则遵循层叠性，否则按选择器优先级判断。 
>
> ![image-20240215125702946.png](https://s2.loli.net/2024/02/15/MOANq15aXnBtuFl.png)

### 优先级

优先级：也叫权重，当一个标签**使用了多种选择器时**，基于不同种类的选择器的**匹配规则**。

```html
<style>
  div {
    color: red;
  }
  .box {
    color: green;
  }
</style>

<div class="box">div 标签</div>
```

#### 基础选择器

规则：选择器**优先级高的样式生效**。

公式：**通配符选择器 < 标签选择器 < 类选择器 < id选择器 < 行内样式 < !important**

​           **（选中标签的范围越大，优先级越低）**

#### 复合选择器-叠加

叠加计算：如果是复合选择器，则需要**权重叠加**计算。

公式：（每一级之间不存在进位）

![1680319646205.png](https://s2.loli.net/2024/02/15/uDgZ6FdlwnfIosv.png)

规则：

* 从左向右依次比较选个数，同一级个数多的优先级高，如果个数相同，则向后比较
* **!important 权重最高**
* 继承权重最低

## Emmet 写法

Emmet写法是代码的**简写**方式，输入缩写 VS Code 会自动生成对应的代码。 

* HTML标签

| 说明         | 标签结构                                         | Emmet             |
| ------------ | ------------------------------------------------ | ----------------- |
| 类选择器     | ``` <div class="box"></div> ```                  | ```标签名.类名``` |
| id选择器     | ```<div id="box"></div>```                       | ```标签名#id名``` |
| 同级标签     | ```<div></div><p></p>```                         | ```div+p```       |
| 父子级标签   | ```<div><p></p></div>```                         | ```div>p```       |
| 多个相同标签 | ```<span>1</span><span>2</span><span>3</span>``` | ```span*3```      |
| 有内容的标签 | ```<div>内容</div>```                            | ```div{内容}```   |



* CSS：大多数简写方式为属性单词的**首字母** 

| 说明      | CSS属性                                          | Emmet         |
| --------- | ------------------------------------------------ | ------------- |
| 宽度      | width                                            | w             |
| 宽度500px | width:500px                                      | w500          |
| 背景色    | background-color                                 | bgc           |
| 多个属性  | widthL200px;height:100px;background-color: #fff; | w200+h100+bgc |



## 背景属性

### 背景图

网页中，使用背景图实现装饰性的图片效果。

* 属性名：**background-image**（bgi）
* 属性值：url(背景图 URL)

```css
div {
  width: 400px;
  height: 400px;

  background-image: url(./images/1.png);
}
```

> 背景图默认有**平铺（复制）效果**。 

### 平铺方式

属性名：**background-repeat**（bgr） 

| 属性值    | 效果         |
| --------- | ------------ |
| no-repeat | 不平铺       |
| repeat    | 平铺（默认） |
| repeat-x  | 水平平铺     |
| repeat-y  | 垂直平铺     |



```css
div {
  width: 400px;
  height: 400px;
  background-color: pink;
  background-image: url(./images/1.png);

  background-repeat: no-repeat;
}
```

### 背景图位置

属性名：**background-position**（bgp）

属性值：水平方向位置 垂直方向位置

* 关键字

| 关键字 | 位置 |
| ------ | ---- |
| left   | 左   |
| right  | 右   |
| center | 居中 |
| top    | 顶部 |
| bottom | 底部 |

* 坐标
  * 水平：正数向右；负数向左
  * 垂直：正数向下；负数向上

```css
div {
  width: 400px;
  height: 400px;
  background-color: pink;
  background-image: url(./images/1.png);
  background-repeat: no-repeat;

  background-position: center bottom;
  background-position: 50px -100px;
  background-position: 50px center;
}
```

> * 关键字取值方式写法，可以颠倒取值顺序
> * 可以只写一个关键字，另一个方向默认为居中；数字只写一个值表示水平方向，垂直方向为居中

### 背景图缩放

作用：设置背景图大小

属性名：**background-size**（bgz）

常用属性值：

* 关键字
  *  cover：等比例缩放背景图片以完全覆盖背景区，可能背景图片部分看不见
  *  contain：等比例缩放背景图片以完全装入背景区，可能背景区部分空白

* 百分比：根据盒子尺寸计算图片大小
* 数字 + 单位（例如：px）

```css
div {
  width: 500px;
  height: 400px;
  background-color: pink;
  background-image: url(./images/1.png);
  background-repeat: no-repeat;
  
  background-size: cover;
  background-size: contain;
}
```

> 工作中，**图片比例与盒子比例相同**，使用 cover 或 contain 缩放背景图效果相同。

### 背景图固定

作用：背景不会随着元素的内容滚动。

属性名：**background-attachment**（bga）

属性值：**fixed**

```css
body {
  background-image: url(./images/bg.jpg);
  background-repeat: no-repeat;
  background-attachment: fixed;
}
```

### 背景复合属性

属性名：**background**（bg）

属性值：背景色 背景图 背景图平铺方式 背景图位置/背景图缩放  背景图固定（**空格隔开各个属性值，不区分顺序**）

```css
div {
  width: 400px;
  height: 400px;

  background: pink url(./images/1.png) no-repeat right center/cover;
}
```

## 显示模式

显示模式是标签（元素）的显示方式。 

![1680320464551.png](https://s2.loli.net/2024/02/15/pedhoUwslm51vMH.png)

作用：布局网页的时候，根据标签的显示模式选择合适的标签摆放内容。 

### 块级元素

特点：

* 独占一行
* 宽度默认是父级的100%
* 添加宽高属性生效

<img src="https://s2.loli.net/2024/02/15/4uFRhGq1xPZkazY.png" alt="1680320578369.png" style="zoom:67%;" />

### 行内元素

特点：

* 一行可以显示多个
* 设置宽高属性不生效
* 宽高尺寸由内容撑开

![1680320583985.png](https://s2.loli.net/2024/02/15/5N8PryJOqBHuFKp.png)

### 行内块元素 

特点：

* 一行可以显示多个
* 设置宽高属性生效
* 宽高尺寸也可以由内容撑开

### 转换显示模式

属性：**display**

| 属性值       | 效果   |
| ------------ | ------ |
| block        | 块级   |
| inline-block | 行内块 |
| inline       | 行内   |


