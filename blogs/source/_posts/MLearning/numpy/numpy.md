---
title: numpy 基本操作
tags: MLearning
categories:
- MLearning
- numpy
---

## numpy基本操作

## 属性与赋值

<!-- more -->

```python
import os
import numpy as np

array = [1, 2, 3, 4, 5]

# ndarray类型的数组是numpy最底层最基本的数据结构
array1 = np.array(array)
print(type(array1))  # 输出: <class 'numpy.ndarray'>

print(array1.dtype)  # 输出: int64

print(array1.size)
# 或者print(np.size(array1)) # 输出: 5, 一共有5个元素

print(array1.itemsize)  # 输出: 8, 每个元素占8个字节

print(array1.ndim)  # 输出: 1, 数组维度

array1.fill(0)
print(array1)  # 输出: [0 0 0 0 0], 所有元素填充0
```

## numpy数组类型

```python
# ***************************同一类型*******************************
# 元素必须是同一类型, 如果不是会自动向下转换
array0 = np.array([1, 2, 3.1, 4, 5])
print(array0)  # 输出: [1.  2.  3.1 4.  5. ]

array0 = np.array([1, 2, 3, 4, "5"])
print(array0)  # 输出: ['1' '2' '3' '4' '5']

array0 = np.array([1, 2, 3, 4, '5'])
print(array0)  # 输出: ['1' '2' '3' '4' '5']

# ***************************不同类型*******************************

array0 = np.array([1, 2, 3, 4, 5, "str"], dtype=np.object)
print(array0)  # [1 2 3 4 5 'str']
print(array0 * 2)  # [2 4 6 8 10 'strstr']


# ***************************类型转换*******************************
array0 = np.array([1, 2, 3, 4, 5])
print(array0.astype(np.float32))  # [1. 2. 3. 4. 5.], 不会改变原来数组类型
print(array0)  # [1 2 3 4 5]

# ***************************索引和切片截取*******************************
array1 = np.array([1, 2, 3, 4, 5])
print(array1[1:3])  # 输出: [2 3], 包括左边索引值, 不包括右边索引值
print(array1[-2:])  # 输出: [4 5], ":"后边不写表示取后边所有索引值
```

## 数组的生成

```
array01 = np.arange(10)
print(array01)  # [0 1 2 3 4 5 6 7 8 9]
array01 = np.arange(2, 20, 2)  # 从2开始到20结束(不包含20), [2, 20), 每个元素数值间隔2的大小
print(array01)  # [ 2  4  6  8 10 12 14 16 18]

array01 = np.arange(2, 20, 2, dtype=np.float32)
print(array01)  # [ 2.  4.  6.  8. 10. 12. 14. 16. 18.]

array01 = np.linspace(0, 10, 10)  # 从0开始到10结束(包含10), [0,10], 等间隔包含10个元素
print(array01)
"""
[ 0.          1.11111111  2.22222222  3.33333333  4.44444444  5.55555556
  6.66666667  7.77777778  8.88888889 10.        ]
"""

array01 = np.logspace(0, 1, 5)  # 从0开始到1结束(包含1), [0,1], 以10为底, 等间隔5个元素
print(array01)
"""
[ 1.          1.77827941  3.16227766  5.62341325 10.        ]
"""

array01 = np.zeros(3)
print(array01)  # [0. 0. 0.]
array01 = np.zeros((3, 3))
print(array01)
"""
[[0. 0. 0.]
 [0. 0. 0.]
 [0. 0. 0.]]
"""
array01 = np.ones((3, 3))
print(array01)
"""
[[1. 1. 1.]
 [1. 1. 1.]
 [1. 1. 1.]]
"""
array01 = np.ones((3, 3)) * 9
print(array01)
"""
[[9. 9. 9.]
 [9. 9. 9.]
 [9. 9. 9.]]
"""

# 构建与原多维数组一样维度的数组
a = np.ones((3, 3))
b = np.ones_like(a)
b = np.zeros_like(a)
print(b)
"""
[[0. 0. 0.]
 [0. 0. 0.]
 [0. 0. 0.]]
"""

# 构建对角矩阵

array01 = np.identity(5)
print(array01)
"""
[[1. 0. 0. 0. 0.]
 [0. 1. 0. 0. 0.]
 [0. 0. 1. 0. 0.]
 [0. 0. 0. 1. 0.]
 [0. 0. 0. 0. 1.]]
"""
```



## numpy 数学运算

### 加减乘法运算

```python
# ***************************数学运算*******************************
array1 = np.array(array)
array2 = array1 + 1
print(array2)  # 输出: [2 3 4 5 6]

# 数组相加, 位数必须相同, 相同位置的元素进行相加
array3 = array2 + array1
print(array3)  # 输出: [ 3  5  7  9 11]

# 数组相乘, 位数必须相同, 对应位置元素进行乘法运算
array4 = array2 * array3 # 与下面等价
array4 = np.multiply(array2, array3)
print(array4)  # 输出: [ 6 15 28 45 66]

print(array4[0])  # 输出: 6

print(array4.shape)  # 输出: (5,)

array5 = np.array([[1, 2, 3], [4, 5, 6]])
print(array5)
print(array5.shape)
"""输出:
[[1 2 3]
 [4 5 6]]
(2, 3)  --> 表示2行, 3列
"""
array6 = np.array([[1, 2, 3],
                   [3, 4, 6],
                   [7, 8, 9]])
print(array6.shape)  # 输出: (3, 3), 3行, 3列
print(array6.ndim)  # 输出: 2, 2维数组
print(array6.size)  # 输出: 9, 总共有9个元素
print(array6)
"""输出
[[1 2 3]
 [3 4 6]
 [7 8 9]]
"""
print(array6[1, 1])  # 4
array6[1, 1] = 10
print(array6[1, 1])  # 10
print(array6[1])  # [ 3 10  6]

print(array6[:, 1])  # [ 2 10  8]
```

### 内积运算
各个元素想乘再求和

```
x = np.array([2, 2])
y = np.array([5, 5])
print(np.dot(x, y)) # 20

x.shape = 2, 1
y.shape = 1, 2
print(np.dot(x, y))
"""
[[10 10]
 [10 10]]
"""
```


### 与或非运算

#### 逻辑与
```
x = np.array([1, 0, 1, 4])
y = np.array([1, 1, 1, 2])
print(x == y) # [ True False  True False]

print(np.logical_and(x, y))  # [ True False  True  True]
```

#### 逻辑或
```
print(np.logical_or(x, y))  # [ True  True  True  True]
```

#### 逻辑非

```
print(np.logical_not(x, y))  # [0 1 0 0]
```


## 浅拷贝与深拷贝

```python
# ***************************浅拷贝与深拷贝*******************************
"""
基本数据类型的特点：直接存储在栈(stack)中的数据.  
引用数据类型的特点：存储的是该对象在栈中引用，真实的数据存放在堆内存里.  
引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体.  
浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象.  
"""

"""
深拷贝, 堆中创建一模一样对象, 栈中创建新的引用
"""
array7 = array.copy()
array7[0] = 100
print(array7)  # [100, 2, 3, 4, 5], 改变新对象的值
print(array)  # [1, 2, 3, 4, 5], 原对象没有改变

"""
浅拷贝, 栈中创建新的引用指向堆中原对象
"""
array8 = array
array8[0] = 100
print(array8)  # [100, 2, 3, 4, 5], 改变原对象的值
print(array)  # [100, 2, 3, 4, 5]
```

## 数据索引

```python
# ***************************数据索引*******************************
"""
np.arange(firstIndex, lastIndex(not included), interval)
"""
tang_array = np.arange(0, 100, 10)
print(tang_array)  # [ 0 10 20 30 40 50 60 70 80 90]

mask = np.array([0, 0, 1, 1, 1, 0, 0, 1, 1, 1], dtype=bool)
print(mask)  # [False False  True  True  True]

print(tang_array[mask])  # [20 30 40 70 80 90]

print(os.path.abspath("."))

random_array = np.random.rand(10)
# [0.90343471 0.40121791 0.3557191  0.84104087 0.1185995  0.64309389 0.49834734 0.19281528 0.54356076 0.85265315]
print(random_array)
mask = random_array > 0.5
# [ True False False  True False  True False False  True  True]
print(mask)
# [0.90343471 0.84104087 0.64309389 0.54356076 0.85265315]
print(random_array[mask])

mask = np.where(random_array > 0.5)
print(mask)  # (array([0, 3, 5, 8, 9]),)
# [0.90343471 0.84104087 0.64309389 0.54356076 0.85265315]
print(random_array[mask])
```

## 数值计算

```python
# ***************************数值计算*******************************
# *******相加*********
tang_array = np.array([[1, 2, 3], [4, 5, 6]])
print(np.sum(tang_array))  # 21, 所有数值相加
print(np.sum(tang_array, axis=0))  # [5 7 9], 竖着相加
# print(tang_array.sum(axis=0))
print(np.sum(tang_array, axis=1))  # [ 6 15], 横着相加
# print(tang_array.sum(axis=1))
print(tang_array.ndim)  # 2, 2维数组

# *******相乘*********
tang_array = np.array([[1, 2, 3], [4, 5, 6]])
print(np.prod(tang_array, axis=0))  # [ 4 10 18], 竖着相乘
print(np.prod(tang_array, axis=1))  # [  6 120], 横着相乘

# *******取最小值, 最大值, 平均值, 标准差, 方差*********
tang_array = np.array([[1, 2, 3], [4, 5, 6]])
print(np.min(tang_array))  # 1
print(np.min(tang_array, axis=0))  # [1 2 3], 竖着最小值
print(np.min(tang_array, axis=1))  # [1 4], 横着最小值

print(np.max(tang_array))  # 6
print(np.mean(tang_array))  # 3.5

print(np.std(tang_array))  # 标准差, 1.707825127659933
print(np.std(tang_array, axis=0))  # 标准差, 竖着算, [1.5 1.5 1.5]
print(np.var(tang_array))  # 方差, 2.9166666666666665

# *******取最小值索引位置*********
print(np.argmin(tang_array))
print(np.argmin(tang_array, axis=0))  # [0 0 0]
print(np.argmin(tang_array, axis=1))  # [0 0]


# ********截取某个范围数据********
tang_array = np.array([[1, 2, 3], [4, 5, 6]])
# 截取2~4之间的数值
print(np.clip(tang_array, 2, 4))
# print(tang_array.clip(2, 4))
"""输出
[[2 2 3]
 [4 4 4]]
"""

# ********四舍五入, 保留1位有效值********
tang_array = np.array([1.2, 3.56, 6.4])
print(tang_array.round())  # [1. 4. 6.]
print(tang_array.round(1))  # [1.2 3.6 6.4]
print(tang_array.round(decimals=1))  # [1.2 3.6 6.4]
```
## 排序

```python
# ********排序********
tang_array = np.array([[1.5, 1.3, 7.5],
                       [5.6, 7.8, 1.2]])
print(np.sort(tang_array))
print(np.sort(tang_array, axis=1))
"""输出
[[1.3 1.5 7.5]
 [1.2 5.6 7.8]]
"""
print(np.argsort(tang_array))
"""排序完后的索引
[[1 0 2]
 [2 0 1]]
"""

# 从[0~10]之间等间隔取10个数据, 包括0和10
tang_array = np.linspace(0, 10, 10)
# [ 0. 1.11111111  2.22222222  3.33333333  4.44444444  5.55555556  6.66666667  7.77777778  8.88888889 10. ]
print(tang_array)
values = np.array([2.5, 6.5, 9.5])
# 只返回搜索插入的位置, 原数组不变
print(np.searchsorted(tang_array, values))  # [3 6 9]

# ********不同行或列同时进行排序********
tang_array = np.array([[1, 0, 6],
                       [1, 7, 0],
                       [2, 3, 1],
                       [2, 4, 0]])
print(tang_array)
# 数组第一列按照从大到小排列, 之后第三列再按照从小到大排列, 总共排列了两次
index = np.lexsort([-1*tang_array[:, 0], tang_array[:, 2]])
print(index)  # [3 1 2 0], 返回lexsort第一个参数排序后index值
print(tang_array[index])
"""输出
[[2 4 0]
 [1 7 0]
 [2 3 1]
 [1 0 6]]
"""
```

## 数组形状
```
tang_array = np.arange(10)
print(tang_array)  # [0 1 2 3 4 5 6 7 8 9]
print(tang_array.shape)  # (10,), 一维数组, 上面一维数组输出可以与下面的二维数组输出形式对比下
tang_array.shape = 2, 5  # 元素个数要保持一致.
print(tang_array)
""" 输出:
[[0 1 2 3 4]
 [5 6 7 8 9]]
"""
tang_array = np.arange(10)
tang_array = tang_array.reshape(2, 5)
print(tang_array)  # reshape 原数组没有改变
"""
[[0 1 2 3 4]
 [5 6 7 8 9]]
"""
```

### 给数组加轴和去掉轴

```
tang_array = np.arange(10)
tang_array = tang_array[np.newaxis, :]
print(tang_array)  # [[0 1 2 3 4 5 6 7 8 9]]
print(tang_array.shape)  # (1, 10), 二维数组, 表示一行十列, 一个样本, 十个元素特征值

tang_array = np.arange(10)
tang_array = tang_array[:, np.newaxis]
print(tang_array)
""" 输出
[[0]
 [1]
 [2]
 [3]
 [4]
 [5]
 [6]
 [7]
 [8]
 [9]]
"""
print(tang_array.shape)  # (10, 1), 二维数组, 表示十行一列, 十个样本, 每个样本一个元素特征值

tang_array = tang_array[:, np.newaxis, np.newaxis]
print(tang_array.shape)  # (10, 1, 1, 1)
tang_array = tang_array.squeeze()  # 压缩多余的轴
print(tang_array.shape)  # (10,)
```

### 矩阵的转置
```
tang_array = tang_array.reshape(2, 5)
#tang_array = tang_array.transpose(), 下面方式更简单
tang_array = tang_array.T
print(tang_array)
""" 输出：
[[0 5]
 [1 6]
 [2 7]
 [3 8]
 [4 9]]
"""
```

### 多维数组的连接和拉长

```
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])
c = np.concatenate((a, b))
c = np.concatenate((a, b), axis=0)  # b作为其它样本拼接在a样本下面
c = np.vstack((a, b))  # 等价于上面
print(c)
"""输出：
[[1 2]
 [3 4]
 [5 6]
 [7 8]]
"""


c = np.concatenate((a, b), axis=1)  # b作为a每个样本其它元素特征拼接在每个样本后面
c = np.hstack((a, b))  # 等价于上面
print(c)
"""输出：
[[1 2 5 6]
 [3 4 7 8]]
"""

d = c.flatten()  # 把多维数组拉长成一维数组
print(d)  # [1 2 5 6 3 4 7 8]
```


## 随机模块

```
1. print(np.random.rand())  # 0.9300429551853078
   print(np.random.random_sample()) # 0.6256746035165891

2. rand_result = np.random.rand(3, 2)
print(rand_result)
""" 输出：
[[0.100757   0.38940102]
 [0.82948206 0.72059258]
 [0.8995502  0.33566056]]
"""

3. rand_result = np.random.randint(10, size=(2, 3), dtype=int)
print(rand_result)
""" 输出：
[[2 5 6]
 [1 3 3]]
"""

rand_result = np.random.randint(0, 10, 3) # 从0到10取三个随机数
print(rand_result) # [0 1 5]
```

### 高斯分布

```
np.set_printoptions(precision=2) # 设置打印数据保留两位有效数据
mu, sigma = 0, 0.1  # 平均值和sigma
gaosifenbu = np.random.normal(mu, sigma, 10)
print(gaosifenbu)
""" 输出：
[ 0.11 -0.1   0.13  0.04 -0.04  0.07  0.1  -0.09 -0.13  0.02]
"""
```

### 洗牌操作

```
tang_array = np.arange(10)
print(tang_array) # [0 1 2 3 4 5 6 7 8 9]
np.random.shuffle(tang_array) # 洗牌操作, 变为乱序
print(tang_array) # [6 4 7 9 1 0 2 5 8 3]
```

### 随机种子

使得每次随机得到的数据都相同

```
np.random.seed(100)
np.set_printoptions(precision=2)
mu, sigma = 0, 0.1
rand_result = np.random.normal(mu, sigma, 10)
print(rand_result) # [-0.17  0.03  0.12 -0.03  0.1   0.05  0.02 -0.11 -0.02  0.03]
tang_array = np.arange(10)
np.random.shuffle(tang_array)
print(tang_array) # [9 7 1 8 5 4 3 2 6 0]
```

## 读写模块

改变当前目录
```
import os
os.chdir("/home/***/learn/python/numpy")
print(os.getcwd()) # /home/***/learn/python/numpy
```

创建tang.txt
```
$ cat tang.txt
1 2 3 4 5 6
2 3 5 7 8 9
```

### python读取文件写法
```
data = []

with open('tang.txt') as f:
    for line in f.readlines():
        fields = line.split()
        cur_data = [float(x) for x in fields]
        data.append(cur_data)
data = np.array(data)
print(data)
""" 输出：
[[1. 2. 3. 4. 5. 6.]
 [2. 3. 5. 7. 8. 9.]]
"""
```

### numpy读取文件

#### txt文件中数据以空格方式隔开
```
data = []
data = np.loadtxt("tang.txt")
print(data)
""" 输出:
[[1. 2. 3. 4. 5. 6.]
 [2. 3. 5. 7. 8. 9.]]
"""
```

#### txt文件中数据以‘,’隔开
tang2.txt文件中数据:
```
$ cat tang2.txt
x,y,z,w,a,b
1,2,3,4,5,6
2,3,5,7,8,9
```
numpy读取文件
 * skiprows = 3:去掉前三行
 * delimiter = ",": 分隔符
 * usecols = (0,1,5): 指定使用哪几列数据

```
data = np.loadtxt("tang2.txt",
                  delimiter=",",
                  skiprows=1,
                  usecols=(0, 1, 3))
print(data)
"""输出:
[[1. 2. 4.]
 [2. 3. 7.]]
"""
```

### numpy保存数据到文件
```
tang_array = np.array([[1, 2, 3], [4, 5, 6]])
np.savetxt("tang3.txt", tang_array, fmt="%d")
np.savetxt("tang4.txt", tang_array, fmt="%.2f")
np.savetxt("tang5.txt", tang_array, fmt="%d", delimiter=",")
```
tang3.txt
```
1 2 3
4 5 6
```
tang4.txt
```
1.00 2.00 3.00
4.00 5.00 6.00
```
tang5.txt
```
1,2,3
4,5,6
```


## 读写array结构
从文件load数据直接是numpy.ndarray格式

### npy格式文件

```
tang_array = np.array([[1, 2, 3], [4, 5, 6]])
np.save("tang_array.npy", tang_array)
tang = np.load("tang_array.npy")
print(tang)
```


### npz格式文件
相当于是把数据以某种方式压缩存储, 目前推荐使用上面 **`npy`** 方式存储

```
tang_array = np.array([[1, 2, 3], [4, 5, 6]])
tang_array2 = np.arange(10)

np.savez("tang_arrayz.npz", a=tang_array, b=tang_array2)
data = np.load("tang_arrayz.npz")

print(data.keys())  # KeysView(<numpy.lib.npyio.NpzFile object at 0x7fe509f96a58>)

print(data["a"])
""" 输出:
[[1 2 3]
 [4 5 6]]
"""

print(data["b"])  # [0 1 2 3 4 5 6 7 8 9]

```


