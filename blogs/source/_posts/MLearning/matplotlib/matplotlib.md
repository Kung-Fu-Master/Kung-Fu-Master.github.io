---
title: 01 matplotlib
tags: MLearning
categories:
- MLearning
- matplotlib
---

python中有很多画图库, 基本上都是用matplotlib进行封装

<!-- more -->

## matplotlib

```
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt

// 查看matplotlib包所在位置
print(mpl.__file__)
// 输出: /usr/local/lib64/python3.6/site-packages/matplotlib/__init__.py

plt.plot([1, 2, 3, 4, 5], [1, 4, 9, 16, 25], "-.", color="r")
#plt.plot([1, 2, 3, 4, 5], [1, 4, 9, 16, 25], linestyle="-.", color="r")

plt.xlabel("xlabel", fontsize=16)
plt.ylabel("ylabel")
plt.show()
#plt.savefig("matplotlib/test01.png")

```

## 线条

### 线条形状
更多线条参考reference: 
https://matplotlib.org/tutorials/index.html
https://matplotlib.org/tutorials/introductory/sample_plots.html
https://matplotlib.org/tutorials/introductory/pyplot.html
https://matplotlib.org/api/_as_gen/matplotlib.lines.Line2D.html#matplotlib.lines.Line2D.set_linestyle
https://matplotlib.org/examples/pylab_examples/multicolored_line.html

| 字符 | 类型 |
| :--------: | :--------: |
| '-' or 'solid' | 实线solid hline |
| '_' | 横线点 |
| '|' | vline |
| '+' | plus |
| '--' or 'dashed' | 虚线dashed line |
| '-.' or 'dashdot' | 虚点线dash-dotted line |
| ':' or 'dotted' | 点线dotted line |
| 'None' or ' ' or '' | draw nothing |
| '.' | 点 |
| ',' | 像素点 |
| 'o' | 圆点 |
| 's' | 正方点 |
| 'p' | 五角点 |
| '*' | star |
| 'h' | 六边形点1 |
| 'H' | 六边形点2 |
| 'D' | 实心菱形点 |
| 'd' | 瘦菱形点 |
| 'x' | filled |
| 'v' | 下三角点 |
| '^' | 上三角点 |
| '<' | 左三角点 |
| '>' | 右三角点 |
| '1' | 下三叉点 |
| '2' | 上三叉点 |
| '3' | 左三叉点 |
| '4' | 右三叉点 |

```
import matplotlib.pylab as plt
markers = [
    '.', ',', 'o', 'v', '^', '<', '>', '1', '2', '3', '4', '8', 's', 'p', 'P',
    '*', 'h', 'H', '+', 'x', 'X', 'D', 'd', '|', '_'
]
descriptions = [
    'point', 'pixel', 'circle', 'triangle_down', 'triangle_up',
    'triangle_left', 'triangle_right', 'tri_down', 'tri_up', 'tri_left',
    'tri_right', 'octagon', 'square', 'pentagon', 'plus (filled)', 'star',
    'hexagon1', 'hexagon2', 'plus', 'x', 'x (filled)', 'diamond',
    'thin_diamond', 'vline', 'hline'
]
x = []
y = []
for i in range(5):
    for j in range(5):
        x.append(i)
        y.append(j)
plt.figure()
for i, j, m, l in zip(x, y, markers, descriptions):
    plt.scatter(i, j, marker=m)
    plt.text(i - 0.15, j + 0.15, s=m + ' : ' + l)
plt.axis([-0.1, 4.8, -0.1, 4.5])
plt.tight_layout()
plt.axis('off')
plt.show()
```
![](line_style.png)

### 线条颜色
reference:
https://matplotlib.org/tutorials/index.html#colors

```
// 线条与颜色分开写
plt.plot([1, 2, 3, 4, 5], [1, 4, 9, 16, 25], "-.", color="r")
// 线条与颜色连在一块写
plt.plot([1, 2, 3, 4, 5], [1, 4, 9, 16, 25], "r-.")
```

## 绘制多条线

```
plt.plot(tang_numpy, tang_numpy, "r--")
plt.plot(tang_numpy, tang_numpy**2, "bs")
// 上面两行等同于以下一行python代码
plt.plot(tang_numpy, tang_numpy, "r--", tang_numpy, tang_numpy**2, "bs")
```

指定线条宽度, 关键点标志
```
x = np.linspace(-10, 10)
y = np.sin(x)

plt.plot(x,
         y,
         linestyle=":",
         linewidth=3.0,
         marker="o",			# 指定x轴值所在位置标志图形
         markerfacecolor="r",	# 标志颜色
         markersize=10,			# 标志大小
         alpha=0.5)				# 透明程度
plt.savefig("matplotlib/line01.png")
plt.show()
```

先画图, 再设置参数
```
x = np.linspace(-10, 10)
y = np.sin(x)
line = plt.plot(x, y)
plt.setp(line,
         color="r",
         linestyle=":",
         linewidth=3.0,
         marker="o",
         markerfacecolor="r",
         markersize=10,
         alpha=0.5)
plt.savefig("matplotlib/line01.png")
plt.show()
```
![](line01.png)
