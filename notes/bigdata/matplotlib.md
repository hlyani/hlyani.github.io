# matplotlib 相关

## 一、matplotlib简介

> matplotlib 是python最著名的绘图库，它提供了一整套和matlab相似的命令API，十分适合交互式地行制图。而且也可以方便地将它作为绘图控件，嵌入GUI应用程序中。
> 它的文档相当完备，并且Gallery页面中有上百幅缩略图，打开之后都有源程序。因此如果你需要绘制某种类型的图，只需要在这个页面中浏览/复制/粘贴一下，基本上都能搞定。
> 在Linux下比较著名的数据图工具还有gnuplot，这个是免费的，Python有一个包可以调用gnuplot，但是语法比较不习惯，而且画图质量不高。
> 而 Matplotlib则比较强：Matlab的语法、python语言、latex的画图质量（还可以使用内嵌的latex引擎绘制的数学公式）。

[](https://blog.csdn.net/Notzuonotdied/article/details/77876080)

[](https://blog.csdn.net/ScarlettYellow/article/details/80458797)

## 二、柱状图

```
plt.bar(X, +Y1, facecolor='#9999ff', edgecolor='white')
```

## 三、折线图

[绘制折线图教程](http://www.jb51.net/article/104916.htm)

##### 1、line chart

```
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(0, 2 * np.pi, 100)
y1, y2 = np.sin(x), np.cos(x)

plt.plot(x, y1)
plt.plot(x, y2)

plt.title('line chart')
plt.xlabel('x')
plt.ylabel('y')

plt.show()
```

##### 2、图例，在plot的时候指定label，然后调用legend方法可以绘制图例。例如：

```
import numpy as np
import matplotlib.pyplot as plt

x = np.linspace(0, 2 * np.pi, 100)
y1, y2 = np.sin(x), np.cos(x)

plt.plot(x, y1, label='y = sin(x)')
plt.plot(x, y2, label='y = cos(x)')
plt.legend()
plt.show()
```

```
legend方法可接受一个loc关键字参数来设定图例的位置，可取值为数字或字符串：
     0: ‘best'
     1: ‘upper right'
     2: ‘upper left'
     3: ‘lower left'
     4: ‘lower right'
     5: ‘right'
     6: ‘center left'
     7: ‘center right'
     8: ‘lower center'
     9: ‘upper center'
     10: ‘center'
```

##### 3、线的样式

* 颜色
  plot方法的关键字参数color(或c)用来设置线的颜色。可取值为：

  ```
  1、颜色名称或简写
    b: blue
    g: green
    r: red
    c: cyan
    m: magenta
    y: yellow
    k: black
    w: white
  2、#rrggbb
  3、(r, g, b) 或 (r, g, b, a)，其中 r g b a 取均为[0, 1]之间
  4、[0, 1]之间的浮点数的字符串形式，表示灰度值。0表示黑色，1表示白色
  ```

* 样式
  plot方法的关键字参数linestyle(或ls)用来设置线的样式。可取值为：

  ```
  -, solid
  --, dashed
  -., dashdot
  :, dotted
  '', ' ', None
  ```

* 粗细
  设置plot方法的关键字参数linewidth(或lw)可以改变线的粗细，其值为浮点数。

  ```
  import numpy as np
  import matplotlib.pyplot as plt
  
  x = np.linspace(0, 2 * np.pi, 100)
  y1, y2 = np.sin(x), np.cos(x)
  
  plt.plot(x, y1, c='r', ls='--', lw=3)
  plt.plot(x, y2, c='#526922', ls='-.')
  plt.show()
  ```

* marker
  以下关键字参数可以用来设置marker的样式：

  ```
  marker
  markeredgecolor 或 mec
  markeredgewidth 或 mew
  markerfacecolor 或 mfc
  markerfacecoloralt 或 mfcalt
  markersize 或 ms
  其中marker可取值为：
  '.': point marker
  ',': pixel marker
  'o': circle marker
  'v': triangle_down marker
  '^': triangle_up marker
  '<': triangle_left marker
  '>': triangle_right marker
  '1': tri_down marker
  '2': tri_up marker
  '3': tri_left marker
  '4': tri_right marker
  's': square marker
  'p': pentagon marker
  '*': star marker
  'h': hexagon1 marker
  'H': hexagon2 marker
  '+': plus marker
  'x': x marker
  'D': diamond marker
  'd': thin_diamond marker
  '|': vline marker
  '_': hline marker
  ```

  例如：

  ```
  import numpy as np
  import matplotlib.pyplot as plt
  
  x = np.linspace(0, 2 * np.pi, 10)
  y1, y2 = np.sin(x), np.cos(x)
  
  plt.plot(x, y1, marker='o', mec='r', mfc='w')
  plt.plot(x, y2, marker='*', ms=10)
  plt.show()
  ```

  另外，marker关键字参数可以和color以及linestyle这两个关键字参数合并为一个字符串。例如：

  ```
  import numpy as np
  import matplotlib.pyplot as plt
  
  x = np.linspace(0, 2 * np.pi, 10)
  y1, y2 = np.sin(x), np.cos(x)
  
  plt.plot(x, y1, 'ro-')
  plt.plot(x, y2, 'g*:', ms=10)
  plt.show()
  ```


##### 4、创建图表
```
plt.figure(1)
```

* left(表示直方图开始位置)
* height(直方图的高度)
* width(直方图宽度)
* yerr(防止直方图触顶)
* color(直方图颜色)

```
rects =plt.bar(left = (1,2,3),height = (1,2,3),color=('r','g','b'),width = 0.5,align="center",yerr=0.000001)

#x1=[0,2,4]
x1=range(3)
y1=[100,230,60]
#x1=range(0,10) 

rects1 =plt.bar(left = x1,height = y1,color=('g'),label=(('no1')),width = 0.5,align="center",yerr=0.000001)
#rects1 =plt.bar(left = (0.2),height = (0.5),color=('g'),label=(('no1')),width = 0.2,align="center",yerr=0.000001)
#rects2 =plt.bar(left = (1),height = (1),color=('r'),label=(('no2')),width = 0.2,align="center",yerr=0.000001)
```

##### 5、直方图上显示具体数字（自动编号）

```
def autolabel(rects):
    for rect in rects:
        height = rect.get_height()
        plt.text(rect.get_x()+rect.get_width()/2., 1.03*height, '%s' % float(height))
autolabel(rects1)
#autolabel(rects2)
```

##### 6、直方图脚注

```
plt.xticks(x1,('a','b','c'))

plt.title('周平均票房'.decode('utf-8'))
plt.xlabel('电影名称'.decode('utf-8'))
plt.ylabel('票房收入(万元)'.decode('utf-8'))

#图注
plt.legend()

plt.show()
```

##### 7、实例

```
# -*- coding: UTF-8 -*-
import matplotlib.pyplot as plt

x1=range(0,10)
y1=[10,13,5,40,30,60,70,12,55,25]
plt.plot(x1,y1,label='A',linewidth=2,color='r',marker='o',markerfacecolor='yellow',markersize=4)
#plt.plot(x1,y1,label='A',linewidth=3,color='r',marker='o', markerfacecolor='b',markersize=12) 

x2=range(0,10)
y2=[5,8,0,30,20,40,50,10,40,15]
plt.plot(x2,y2,label='B',color='black',ls='-.')

x3=range(0,10)
y3=[1,60,10,3,2,40,20,10,40,5]
plt.plot(x3,y3,label='C',color='blue')

plt.xlabel('time')
plt.ylabel('income')
plt.title('my Graph')
plt.legend()
plt.show()
```

