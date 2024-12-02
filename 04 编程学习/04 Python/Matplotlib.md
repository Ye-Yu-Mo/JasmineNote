---
title: Python可视化之Matplotlib
date: 2023-12-27 19:00:03
tags: [python,matplotlib]
categories: [python]
---

## Matplotlib与可视化分析

我们之前对数据的处理与分析，其实最终还是要利用可视化工具进行更加直观的输出

我们开业通过

```python
pip install matplotlib
```

命令来安装对应的模块

### 简单图形的绘制

我们可以通过matplotlib的子模块pyplot来进行平面图像的绘制，比较便捷，而且输出格式也更加多样化

我们可以给他起个名字叫plt

例如

```python
import math
import matplotlib.pyplot as plt

nbSamples = 256
xRange = (-math.pi, math.pi)
x, y = [], []

for n in range(nbSamples):
    ratio = (n + 0.5) / nbSamples
    x.append(xRange[0] + (xRange[1] - xRange[0]) * ratio)
    y.append(math.sin(x[-1]))

plt.plot(x, y)
plt.show()
```

结果如下

![屏幕截图 2023-12-27 191126.png](https://s2.loli.net/2023/12/27/O9ikw6tKDpMfvxX.png)

这里最复杂的其实是对x和y函数的构造，实际上关于绘图的代码是由最后两行完成的，plot是构建图像，show则是显示图像

对于各种复杂的数据计算和处理，实际上可以结合numpy的内容辅助处理

我们也可以对图像中线条的属性，例如color选项可以选择图像的颜色，linewidth可以选择线条的宽度，linestyle可以选择线条的样式

这里我们给出常用的颜色

| 字母 | 颜色   |
| ---- | ------ |
| r    | 红色   |
| b    | 蓝色   |
| m    | 紫色   |
| k    | 黑色   |
| g    | 绿色   |
| c    | 青色   |
| y    | 土黄色 |
| w    | 白色   |

### pylot的高级功能

#### 添加图例与注释

例如

```python
nbSamples = 128

x = np.linspace(-np.pi, np.pi, nbSamples)
y1 = np.sin(x)
y2 = np.cos(x)

plt.plot(x, y1, color='g', linewidth=4, linestyle='--', label=r'$y=sin(x)')
plt.plot(x, y2, '*', markersize=8, markerfacecolor='r', markeredgecolor='k', label=r'$y=cos(x)')

plt.legend(loc='best')
plt.show()

```

![屏幕截图 2023-12-27 192357.png](https://s2.loli.net/2023/12/27/ZkHJ7afKi3spMtQ.png)

这里我们加入了属性标签，legend是放在了最佳位置，也可以进行设置，例如，upper right,upper left,lower left,center left 等

对于这里的美元符号则是对应着LaTeX公式，可以访问可以更加高效的描述数学公式

除此之外，我们也可以设置显示一些其他的文本提示，例如坐标提示等

例如

```python
plt.text(a, b, (a, b), ha = 'center', va = 'bottom', fontsize = 10)
```

设置标题以及坐标轴

```python
plt.title()
plt.xlabel()
```

Matplotlib对于中文支持并不友好，在Windows平台下，我们需要设置如下参数显示中文文本

```python
plt.rcParams['font.sans-serif']=['SimHei'] # 用于正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False # 设置正常显示政府和
```

这里的sim指的是simple简体中文，hei表示黑体

关于众多子图的内容，还有更多的matplotlib内容，还请阅读官方文档，我们对于基本的图像绘制了解到此基本就足以日常使用了，如果需要更多的方法，官方文档绝对是一个更好的选择

我们的Python基础部分也告一段落，之后会陆续推出其他内容，还请大家继续支持
