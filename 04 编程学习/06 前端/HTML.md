---
title: HTML基础
date: 2024-02-02 16:20:19
tags: [Web,HTML]
categories: [Web]
---

# HTML 基础

HTML的意思就是超文本标记语言（HyperText Markup Language），这里的超文本的意思就是多媒体内容，有文本、图片、音视频等内容，这里的标记就是使用代码的形式，使用尖括号的标签进行标记的意思，例如

![1680242502155.png](https://s2.loli.net/2024/02/02/q7JCtV6sNBackx5.png)

## 标签结构

* 标签要成对出现，中间包裹内容
* <>里面放英文字母（标签名）
* 结束标签比开始标签多 /
* 标签分类：双标签和单标签

```html
<strong>需要加粗的文字<strong>
<br>
<hr>
```

## HTML骨架

* html：整个网页
* head：网页头部，用来存放给浏览器看的信息，例如 CSS
  * title：网页标题
* body：网页主体，用来存放给用户看的信息，例如图片、文字

```html
<html>
  <head>
    <title>网页标题</title>
  </head>
  <body>
    网页主体
  </body>
</html>
```

> VS Code 可以快速生成骨架：在 HTML 文件（.html）中，!（英文）配合 Enter / Tab 键

## 标签的关系

* 父子关系（嵌套关系）：子级标签换行且缩进（Tab键）

  ![1680255766672.png](https://s2.loli.net/2024/02/02/WEH4lCwmeVjYQhS.png)

* 兄弟关系（并列关系）：兄弟标签换行要对齐

  ![1680255775150.png](https://s2.loli.net/2024/02/02/zXvoWkfKdBniwcE.png)

  

## 注释

概念：注释是对代码的解释和说明，能够提高程序的可读性，方便理解、查找代码。

注释不会再浏览器中显示。

在 VS Code 中，**添加 / 删除**注释的快捷键：**Ctrl + /** 

```html
<!-- 注释 -->
```

## 标题标签

```html
<h1>一级标题</h1>
<h2>二级标题</h2>
<h3>三级标题</h3>
<h4>四级标题</h4>
<h5>五级标题</h5>
<h6>六级标题</h6>
```

显示特点：

* 文字加粗
* 字号逐渐减小
* 独占一行（换行）

> 1. h1 标签在一个网页中只能用一次，用来放新闻标题或网页的 logo
> 2. h2 ~ h6 没有使用次数的限制

## 段落标签

一般用在新闻段落、文章段落、产品描述信息等等。 

```html
<p>段落</p>
```

显示特点：

* 独占一行
* 段落之间存在间隙

## 换行和水平线

* 换行：br
* 水平线：hr

```html
<br>
<hr>
```

## 文本格式化标签

作用：为文本添加特殊格式，以突出重点。常见的文本格式：加粗、倾斜、下划线、删除线等。 

| 标签名   | 效果          |
| -------- | ------------- |
| strong/b | **加粗**      |
| em/i     | *倾斜*        |
| ins/u    | <u>下划线</u> |
| del/s    | <s>删除线</s> |

> strong、em、ins、del 标签自带强调含义（语义），比较常用 

## 图像标签

作用：在网页中插入图片

```html
<img  src="图片的 URL">
```

src用于指定图像的位置和名称，是 \<img\> 的必须属性。 

### 图像属性

| 属性   | 作用     | 说明                 |
| ------ | -------- | -------------------- |
| alt    | 替换文本 | 图片无法显示时的文字 |
| title  | 提示文本 | 悬停时的提示文字     |
| width  | 图片宽度 | 数值，无单位         |
| height | 图片高度 | 数值，无单位         |

这里的宽度和高度我们一般不在这里设置，等到我们学习CSS时会深入了解

### 属性语法

* 属性名="属性值"
* 属性写在尖括号里面，标签名后面，标签名和属性之间用空格隔开，不区分先后顺序

![1680314195951.png](https://s2.loli.net/2024/02/02/wAHDW49uTJfUosQ.png)

## 路径

概念：路径指的是查找文件时，从**起点**到**终点**经历的**路线**。 

路径分类：

* 相对路径：从当前文件位置出发查找目标文件
* 绝对路径：从盘符出发查找目标文件
  * Windows 电脑从盘符出发
  * Mac 电脑从根目录出发

### 相对路径

查找方式：从**当前文件位置**出发查找目标文件

特殊符号：

* **/** 表示进入某个文件夹里面          → 文件夹名/
* **. ** 表示当前文件所在文件夹           → ./
* **..** 表示当前文件的上一级文件夹 → ../   

### 绝对路径

查找方式：Windows 电脑从盘符出发；Mac 电脑从根目录（/）出发

```html
<img  src="C:\images\mao.jpg">
```

## 超链接标签

作用：点击跳转到其他页面。 

```html
<a href="https://www.baidu.com">跳转到百度</a>
```

**href 属性值是跳转地址，是超链接的必须属性。**

超链接默认是在当前窗口跳转页面，添加 **target="_blank"** 实现**新窗口**打开页面。

拓展：开发初期，不确定跳转地址，则 href 属性值写为 **#**，表示**空链接**，页面不会跳转，在当前页面刷新一次。

```html
<a href="https://www.baidu.com/">跳转到百度</a>

<!-- 跳转到本地文件：相对路径查找 --> 
<!-- target="_blank" 新窗口跳转页面 --> 
<a href="./01-标签的写法.html" target="_blank">跳转到01-标签的写法</a>

<!-- #表示空连接，不会跳转，后期可以更换为具体链接 -->
<a href="#">空链接</a>
```

## 音频

```html
<audio src="音频的 URL"></audio>
```

| 属性     | 作用             | 说明          |
| -------- | ---------------- | ------------- |
| src      | 音频源           | MP3、Ogg、wav |
| controls | 显示音频控制面板 | 默认禁用      |
| loop     | 循环播放         | 默认禁用      |
| autoplay | 自动播放         | 默认禁用      |



> 书写 HTML5 属性时，如果属性名和属性值相同，可以简写为一个单词。 

```html
<!-- 在 HTML5 里面，如果属性名和属性值完全一样，可以简写为一个单词 -->
<audio src="./media/music.mp3" controls loop autoplay></audio>
```

## 视频

```html
<video src="视频的 URL"></video>
```

常用属性

| 属性     | 作用         | 说明               |
| -------- | ------------ | ------------------ |
| src      | 视频源       | MP4、WebM、Ogg     |
| controls | 视频控制面板 |                    |
| loop     | 循环播放     |                    |
| muted    | 静音播放     |                    |
| autoplay | 自动播放     | 支持静音的自动播放 |



```html
<!-- 在浏览器中，想要自动播放，必须有 muted 属性 -->
<video src="./media/vue.mp4" controls loop muted autoplay></video>
```


## 列表

列表是布局内容排列整齐的区域，分为无序列表、有序列表、定义列表。

### 无序列表

无序列表是布局排列整齐的**不需要规定顺序**的区域。

标签：ul 嵌套 li，ul 是无序列表，li 是列表条目。

例如

```html
<ul>
  <li>第一项</li>
  <li>第二项</li>
  <li>第三项</li>
  ……
</ul>
```

> * ul 标签里面只能包裹 li 标签
> * li 标签里面可以包裹任何内容

### 有序列表

有序列表是布局排列整齐的**需要规定顺序**的区域。

标签：ol 嵌套 li，ol 是有序列表，li 是列表条目。

例如

```html
<ol>
  <li>第一项</li>
  <li>第二项</li>
  <li>第三项</li>
  ……
</ol>
```

> * ol 标签里面只能包裹 li 标签
> * li 标签里面可以包裹任何内容

### 定义列表

标签：dl 嵌套 dt 和 dd，dl 是定义列表，dt 是定义列表的标题，dd 是定义列表的描述 / 详情。

```html
<dl>
  <dt>列表标题</dt>
  <dd>列表描述 / 详情</dd>
   ……
</dl>
```

![1680315652403.png](https://s2.loli.net/2024/02/04/ibFUadX7x5IepAL.png)

> * dl 里面只能包含dt 和 dd
> * dt 和 dd 里面可以包含任何内容

## 表格

网页中的表格与 Excel 表格类似，用来展示数据。 

![1680315690740.png](https://s2.loli.net/2024/02/04/XwZvtSR2erMQK4U.png)

### 基本使用

标签：table 嵌套 tr，tr 嵌套 td / th。 

| 标签名 | 说明 |
| ------ | ---- |
| table  | 表格 |
| tr     | 行   |
| th     | 表头 |
| td     | 内容 |



> 提示：在网页中，**表格默认没有边框线**，使用 **border 属性**可以为表格添加边框线。 

```html
<table border="1">
  <tr>
    <th>姓名</th>
    <th>语文</th>
    <th>数学</th>
    <th>总分</th>
  </tr>
  <tr>
    <td>张三</td>
    <td>99</td>
    <td>100</td>
    <td>199</td>
  </tr>
  <tr>
    <td>李四</td>
    <td>98</td>
    <td>100</td>
    <td>198</td>
  </tr>
  <tr>
    <td>总结</td>
    <td>全市第一</td>
    <td>全市第一</td>
    <td>全市第一</td>
  </tr>
</table>
```

### 表格结构标签-了解

用表格结构标签把内容划分区域，可以让表格结构更清晰，语义更清晰。

| 标签名 | 含义     | 说明     |
| ------ | -------- | -------- |
| thead  | 表格头部 | 头部内容 |
| tbody  | 表格主体 | 主要内容 |
| tfoot  | 表格底部 | 汇总信息 |



> 表格结构标签可以省略。

### 合并单元格

作用：将多个单元格合并成一个单元格，以合并同类信息。 

![1680315812998.png](https://s2.loli.net/2024/02/04/4zwVbyFWxfne1Td.png)

合并单元格的步骤：

1. 明确合并的目标
2. 保留**最左最上**的单元格，添加属性（取值是**数字**，表示需要**合并的单元格数量**）
   * **跨行合并**，保留最上单元格，添加属性 **rowspan**
   * **跨列合并**，保留最左单元格，添加属性 **colspan**
3. 删除其他单元格

```html
<table border="1">
  <thead>
    <tr>
      <th>姓名</th>
      <th>语文</th>
      <th>数学</th>
      <th>总分</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>张三</td>
      <td>99</td>
      <td rowspan="2">100</td>
      <td>199</td>
    </tr>
    <tr>
      <td>李四</td>
      <td>98</td>
      <!-- <td>100</td> -->
      <td>198</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td>总结</td>
      <td colspan="3">全市第一</td>
      <!-- <td>全市第一</td>
      <td>全市第一</td> -->
    </tr>
  </tfoot>
</table>
```

> 不能跨表格结构标签合并单元格（thead、tbody、tfoot）。

## 表单

表单是用于收集用户信息。

使用场景：

* 登录页面
* 注册页面
* 搜索区域

### input 标签

input 标签 type 属性值不同，则功能不同。 

```html
<input type="..." >
```

| type属性 | 说明     |
| -------- | -------- |
| text     | 文本框   |
| password | 密码框   |
| radio    | 单选框   |
| checkbox | 多选框   |
| file     | 上传文件 |



### input 标签占位文本 

占位文本：提示信息，文本框和密码框都可以使用。 

```html
<input type="..." placeholder="提示信息">
```

### 单选框

常用属性

| 属性    | 作用     | 说明                     |
| ------- | -------- | ------------------------ |
| name    | 控件名称 | 同组只能选择一个（单选） |
| checked | 默认选中 | 默认选择                 |



```html
<input type="radio" name="gender" checked> 男
<input type="radio" name="gender"> 女
```

> name 属性值自定义。

### 上传文件 

默认情况下，文件上传表单控件只能上传一个文件，添加 multiple 属性可以实现文件多选功能。

```html
<input type="file" multiple>
```

### 多选框

多选框也叫复选框，默认选中：checked。

```html
<input type="checkbox" checked> 前端
```

### 下拉菜单

![1680316175031.png](https://s2.loli.net/2024/02/04/RtYvwjO9FxZCsN2.png)

标签：select 嵌套 option，select 是下拉菜单整体，option是下拉菜单的每一项。

```html
<select>
  <option>北京</option>
  <option>上海</option>
  <option>广州</option>
  <option>深圳</option>
  <option selected>武汉</option>
</select>
```

> 默认显示第一项，**selected** 属性实现**默认选中**功能。

### 文本域

作用：多行输入文本的表单控件。 

![1680316238194.png](https://s2.loli.net/2024/02/04/XHGIZusqiapAC5n.png)

```html
<textarea>默认提示文字</textarea>
```

> * 实际开发中，使用 CSS 设置 文本域的尺寸
> * 实际开发中，一般禁用右下角的拖拽功能

### label 标签 

作用：网页中，某个标签的说明文本。 

![1680316296894.png](https://s2.loli.net/2024/02/04/4KHjMluWwGZ3zNY.png)

用 label 标签绑定文字和表单控件的关系，增大表单控件的点击范围。 

![1680316314721.png](https://s2.loli.net/2024/02/04/VF7BrMxLXN9lp2T.png)

* 写法一
  * label 标签只包裹内容，不包裹表单控件
  * 设置 label 标签的 for 属性值 和表单控件的 id 属性值相同

```html
<input type="radio" id="man">
<label for="man">男</label>
```

* 写法二：使用 label 标签包裹文字和表单控件，不需要属性 

```html
<label><input type="radio"> 女</label>
```

> 支持 label 标签增大点击范围的表单控件：文本框、密码框、上传文件、单选框、多选框、下拉菜单、文本域等等。 

### 按钮

```html
<button type="">按钮</button>
```

![1680316426088.png](https://s2.loli.net/2024/02/04/SsUwA2i14vFZJm8.png)

| type属性 | 说明                       |
| -------- | -------------------------- |
| submit   | 提交，默认功能             |
| reset    | 重置，需要配合form标签使用 |
| button   | 按钮，配合JavaScript使用   |



```html
<!-- form 表单区域 -->
<!-- action="" 发送数据的地址 -->
<form action="">
  用户名：<input type="text">
  <br><br>
  密码：<input type="password">
  <br><br>

  <!-- 如果省略 type 属性，功能是 提交 -->
  <button type="submit">提交</button>
  <button type="reset">重置</button>
  <button type="button">普通按钮</button>
</form>
```

> 按钮需配合 form 标签（表单区域）才能实现对应的功能。

## 语义化

### 无语义的布局标签 

语义标签是用于布局网页（划分网页区域，摆放内容）

* div：独占一行
* span：不换行

```html
<div>div 标签，独占一行</div>
<span>span 标签，不换行</span>
```

### 有语义的布局标签

| 标签名  | 语义       |
| ------- | ---------- |
| header  | 网页头部   |
| nav     | 网页导航   |
| footer  | 网页底部   |
| aside   | 网页侧边栏 |
| section | 网页区块   |
| article | 网页文章   |



## 字符实体

字符实体是由于HTML语言使用了这些符号描述，当我们需要显示空格、大于号、小于号时，就需要使用后面的符号进行替代

| 显示 | 描述   | 符号    |
| ---- | ------ | ------- |
|      | 空格   | \&nbsp; |
| <    | 小于号 | \&lt;   |
| >    | 大于号 | \&gt;   |

