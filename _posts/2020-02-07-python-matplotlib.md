---
layout: post
title: "Static Data Analysis with Matplotlib"
date: 2020-02-07 15:23:11
image: '/assets/img/'
description: '使用 Python Matplotlib 做静态数据分析'
main-class: python
color: 
tags: 
 - tools
 - python
 - numpy
 - matplotlib
 - conda
 - scipy
categories: 
 - python
twitter_text: 'Static Data Analysis with Matplotlib'
introduction: 'Static Data Analysis with Matplotlib'
---

# 前言

**[NumPy][numpy]** 是 Python 中的一个用于科学计算的基础包

具备以下特性

~~~
有强大的 N 维数组处理能力
精度计算的能力
可以整合 C/C++ and Fortran 的代码
完成线性代数，傅立叶变换和随机数的生成等工作
~~~

它有具备优秀的科学计算能力，NumPy 也可以用于多维数据的生成，可以支持各种类型的数据，这使得它可以方便地与各类数据库相集成

**[SciPy][scipy]** 也是 Python 中的一个用于科学计算的基础包

>SciPy (pronounced “Sigh Pie”) is a Python-based ecosystem of open-source software for mathematics, science, and engineering

**[Matplotlib][matplotlib]** 是 Python 中的一个 2D 绘图库

>Matplotlib is a Python 2D plotting library which produces publication quality figures in a variety of hardcopy formats and interactive environments across platforms

这里分享一下使用 **[NumPy][numpy]、[SciPy][scipy]、[Matplotlib][matplotlib]** 进行描述性统计分析的简单用法

>**Tip:** 当前的版本为 **Python 3.8.1**, **conda 4.8.2**

---

# 操作

## 系统环境

~~~
(calc3.8) ➜  calc sw_vers 
ProductName:	Mac OS X
ProductVersion:	10.15.3
BuildVersion:	19D76
(calc3.8) ➜  calc 
(calc3.8) ➜  calc python --version
Python 3.8.1
(calc3.8) ➜  calc conda --version
conda 4.8.2
(calc3.8) ➜  calc 
~~~

## 安装基础包

使用 **`conda install <packagename>`** 的方式对 **`matplotlib、numpy、scipy`** 进行安装

~~~
(calc3.8) ➜  calc conda list | grep -iE '(numpy|scipy|matplot)'
matplotlib                3.1.3                    py38_0  
matplotlib-base           3.1.3            py38h9aa3819_0  
numpy                     1.18.1           py38h7241aed_0  
numpy-base                1.18.1           py38h6575580_1  
scipy                     1.3.1            py38h1410ff5_0  
(calc3.8) ➜  calc 
~~~

**[Conda][conda]** 是一个用于环境版本管理的软件，适用于很多类型的语言

>Package, dependency and environment management for any language—Python, R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, FORTRAN, and more.

## 准备测试数据

~~~
(calc3.8) ➜  calc vim abc.txt 
(calc3.8) ➜  calc cat abc.txt 
202001011950:2,3,4,5,6,1,3,1,2,3,5,5
202001021950:3,2,1,3,4,5,2
202001031950:7,8,9,9,3,2,4,1
202001011150:2,3,2,1,8,6,4,9,10
202001021050:2,3,1,6,7,9,0,12
202001031750:1,2,1,10,2,3,5,6
(calc3.8) ➜  calc 
~~~

这个数据符合如下规范

* 这是一个文本
* "`:`" 将每行分隔成两个部分
* 第一个部分为 yyyymmddHHMM 的文本
* 第二个部分为一系列的数字
* 顺序无关(由程序来自动排序)


## 上代码

直接在每行代码中进行了注释

~~~
import numpy as np  # 导入 numpy 包
from scipy import stats  # 导入 scipy 中的 stats 包
import matplotlib.pyplot as plt  # 导入 matplotlib.pyplot 包

source = {}  # 定义一个用于存放源数据的字典
res = {}  # 定义一个用于存放目标数据的字典

for i in open("abc.txt"):  # 逐行读取 abc.txt 中的内容,  abc.txt 为一个普通文本，其中每行的数据格式为 yyyymmddHHMM:int,int,int,int 例如： 202001011950:2,3,4,5,6,1,3,1,2,3,5,5
    tag = i.split(':')[0]  # 将 x 坐标数据取出
    dataList = i.split(':')[1]  # 将其余数据取出作为要统计的内容集
    source[tag] = [int(j) for j in dataList.split(',')]  # 将内容集从 str 转换为 int 类型， 然后存入 source 字典中
    res[tag] = {     # 目标字典里以 x 坐标作为 key, 存入如下数据
        "num": len(source[tag]),          # 数据集的长度
        "avg": np.mean(source[tag]),   # 数据集的均值
        "median": np.median(source[tag]),   # 数据集的中位数
        "pop": stats.mode(source[tag])[0][0],  # 数据集的众数
        "ptp": np.ptp(source[tag]),  # 数据集的极差
        "var": np.var(source[tag]),  # 数据集的方差
        "std": np.std(source[tag]),  # 数据集的标准差
        "kurtosis": stats.kurtosis(source[tag]),  # 数据集的峰度
        "skew": stats.skew(source[tag]),  # 数据集的偏度
        "per25": np.percentile(source[tag], 25),  # 数据集的25分位数
        "per75": np.percentile(source[tag], 75),  # 数据集的75分位数
        "min": np.min(source[tag]),  # 数据集的最小值
        "max": np.max(source[tag])   # 数据集的最大值
    }

# print(res)
# print(res.keys())

res_array = list(res.keys())  # 将结果集的 key 都取出，转化为 list
res_array.sort()  # 将 list 进行排序

# 定义出 list 用于存放各纬度数据

x = []
num = []
avg = []
median = []
pop = []
ptp = []
var = []
std = []
kurtosis = []
skew = []
per25 = []
per75 = []
min = []
max = []

# 生成以下 list 用于存放结果，为绘图做填充，分别对应着上面的各纬度

for i in res_array:
    x.append(i)
    num.append(res[i]["num"])
    avg.append(res[i]["avg"])
    median.append(res[i]["median"])
    pop.append(res[i]["pop"])
    ptp.append(res[i]["ptp"])
    var.append(res[i]["var"])
    std.append(res[i]["std"])
    kurtosis.append(res[i]["kurtosis"])
    skew.append(res[i]["skew"])
    per25.append(res[i]["per25"])
    per75.append(res[i]["per75"])
    min.append(res[i]["min"])
    max.append(res[i]["max"])


rotation = 10  # xticks 的旋转角度

plt.figure()  # 绘制一张图
plt.subplot(2, 3, 1)  # 切分成2行3例，在第1个区域绘制如下内容
plt.xticks(rotation=rotation, fontsize='xx-small')  # 将 x 轴的标注旋转10度，使用超小字体
plt.boxplot([source[i] for i in x], labels=x)  # 使用 source 中的数据集绘制箱体图
# plt.figure()
plt.subplot(2, 3, 2)  # 切分成2行3例，在第2个区域绘制如下内容
plt.xticks(rotation=rotation, fontsize='xx-small')  # 将 x 轴的标注旋转10度，使用超小字体
plt.plot(x, max, '^-g', label='max')  # 绘制最大值随 x 的变化曲线，使用上三角号作为marker的绿色实线
plt.plot(x, median, '*--b', label='median')  # 绘制中位数随 x 的变化曲线，使用星角号作为marker的蓝色虚线
plt.plot(x, avg, 'o--r', label='avg')  # 绘制均值随 x 的变化曲线，使用圆形作为marker的红色虚线
plt.plot(x, pop, '|:c', label='pop')  # 绘制众数随 x 的变化曲线，使用竖线作为marker的青色点线
plt.plot(x, ptp, 'p:y', label='ptp')  # 绘制极差随 x 的变化曲线，使用多边形作为marker的黄色点线
plt.plot(x, min, 'H--k', label='min')  # 绘制最小值随 x 的变化曲线，使用多边形作为marker的黑色虚线
plt.legend()  # 绘制图例
# plt.figure()
plt.subplot(2, 3, 4)  # 切分成2行3例，在第4个区域绘制如下内容
plt.xticks(rotation=rotation, fontsize='xx-small')   # 将 x 轴的标注旋转10度，使用超小字体
plt.plot(x, num, '-.', label='num', color='green', marker='o')   # 绘制数值个数随 x 的变化曲线，使用圆形作为marker的绿色虚点线
# plt.text(x, num, s=num)
for i in x:
    plt.text(i, res[i]["num"], '%d' % res[i]["num"], ha='center', va='bottom', fontsize=9)   # 在 marker 上添加数值作为标记
plt.legend()  # 绘制图例
plt.subplot(2, 3, 3)  # 切分成2行3例，在第3个区域绘制如下内容
# plt.figure()
plt.xticks(rotation=rotation, fontsize='xx-small')   # 将 x 轴的标注旋转10度，使用超小字体
plt.plot(x, var, label='var', ls='-.')   # 绘制方差随 x 的变化曲线，使用随机颜色的虚点线
for i in x:
    plt.text(i, res[i]["var"], '%.4f' % res[i]["var"], ha='center', va='bottom', fontsize=9)  # 在 marker 上添加数值作为标记
plt.plot(x, std, label='std', ls='--')   # 绘制标准差随 x 的变化曲线，使用随机颜色的虚线
for i in x:
    plt.text(i, res[i]["std"], '%.4f' % res[i]["std"], ha='center', va='top', fontsize=9)  # 在 marker 上添加数值作为标记
plt.legend()  # 绘制图例
# plt.figure()
plt.subplot(2, 3, 6)   # 切分成2行3例，在第6个区域绘制如下内容
plt.xticks(rotation=rotation, fontsize='xx-small')    # 将 x 轴的标注旋转10度，使用超小字体
plt.plot(x, kurtosis, ':r', label='kurtosis')   # 绘制峰度随 x 的变化曲线，使用红颜色的虚点
for i in x:
    plt.text(i, res[i]["kurtosis"], '%.4f' % res[i]["kurtosis"], ha='center', va='top', fontsize=9)   # 在 marker 上添加数值作为标记
plt.plot(x, skew, '-.y', label='skew')   # 绘制偏度随 x 的变化曲线，使用黄颜色的虚点线
for i in x:
    plt.text(i, res[i]["skew"], '%.4f' % res[i]["skew"], ha='center', va='bottom', fontsize=9)   # 在 marker 上添加数值作为标记
plt.legend()   # 绘制图例
plt.show()   # 绘图
~~~

## 运行

![python](/assets/img/python/python01.png)

使用 **[NumPy][numpy]** 和 **[SciPy][scipy]** 结合 **[Matplotlib][matplotlib]** 可以很方便地进行数据统计 

---

# 总结

常见的数据分析函数都集成到了 **[NumPy][numpy]** 和 **[SciPy][scipy]** 两个包中

这里结合 **[Matplotlib][matplotlib]** 快速生成了可视化的图表

后面再结合一些前端的库可以生成一些交互式的数据图表

作为数据可视化的基础入门

* TOC
{:toc}


---


[numpy]:https://numpy.org/
[scipy]:https://www.scipy.org/
[matplotlib]:https://matplotlib.org/
[conda]:https://docs.conda.io/en/latest/
[statistics]:https://docs.scipy.org/doc/numpy/reference/routines.statistics.html
[scipy_stats]:https://docs.scipy.org/doc/scipy/reference/stats.html
[ref]:https://zhuanlan.zhihu.com/p/57241112
[scipy_doc]:https://docs.scipy.org/doc/

