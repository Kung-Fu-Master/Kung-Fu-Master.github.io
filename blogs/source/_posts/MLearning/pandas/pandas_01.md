---
title: 01 pandas 数据读取与展示
tags: MLearning
categories:
- MLearning
- pandas
---

pandas是把numpy包里的很多命令整合一下使其处理数据更方便

## pandas读取csv

<!-- more -->

``` python
import pandas as pd
import os
os.chdir("/home/***/learn/python/pandas")
print(os.getcwd()) # /home/***/learn/python/pandas

df = pd.read_csv("data/titanic.csv")

print(help(pd.read_csv))


print(df.head(6)) # 输出前6行
""" 输出:
   PassengerId  Survived  Pclass                                               Name  ...            Ticket     Fare  Cabin  Embarked
0            1         0       3                            Braund, Mr. Owen Harris  ...         A/5 21171   7.2500    NaN         S
1            2         1       1  Cumings, Mrs. John Bradley (Florence Briggs Th...  ...          PC 17599  71.2833    C85         C
2            3         1       3                             Heikkinen, Miss. Laina  ...  STON/O2. 3101282   7.9250    NaN         S
3            4         1       1       Futrelle, Mrs. Jacques Heath (Lily May Peel)  ...            113803  53.1000   C123         S
4            5         0       3                           Allen, Mr. William Henry  ...            373450   8.0500    NaN         S
5            6         0       3                                   Moran, Mr. James  ...            330877   8.4583    NaN         Q

[6 rows x 12 columns]
"""


print(df.tail(6)) # 输出后6行

"""输出:
     PassengerId  Survived  Pclass                                      Name     Sex   Age  SibSp  Parch      Ticket    Fare Cabin Embarked
885          886         0       3      Rice, Mrs. William (Margaret Norton)  female  39.0      0      5      382652  29.125   NaN        Q
886          887         0       2                     Montvila, Rev. Juozas    male  27.0      0      0      211536  13.000   NaN        S
887          888         1       1              Graham, Miss. Margaret Edith  female  19.0      0      0      112053  30.000   B42        S
888          889         0       3  Johnston, Miss. Catherine Helen "Carrie"  female   NaN      1      2  W./C. 6607  23.450   NaN        S
889          890         1       1                     Behr, Mr. Karl Howell    male  26.0      0      0      111369  30.000  C148        C
890          891         0       3                       Dooley, Mr. Patrick    male  32.0      0      0      370376   7.750   NaN        Q
"""


print(df.info())
"""输出:
<class 'pandas.core.frame.DataFrame'>      // 读取csv返回DataFrame类型数据
RangeIndex: 891 entries, 0 to 890          // csv总共有891行数据
Data columns (total 12 columns):           // csv总共有12列数据
 #   Column       Non-Null Count  Dtype  
---  ------       --------------  -----  
 0   PassengerId  891 non-null    int64  
 1   Survived     891 non-null    int64  
 2   Pclass       891 non-null    int64  
 3   Name         891 non-null    object 
 4   Sex          891 non-null    object 
 5   Age          714 non-null    float64  // 只有714行数据, 需要做数据填充
 6   SibSp        891 non-null    int64  
 7   Parch        891 non-null    int64  
 8   Ticket       891 non-null    object   // 含有字符串, 成为object类型
 9   Fare         891 non-null    float64
 10  Cabin        204 non-null    object 
 11  Embarked     889 non-null    object 
dtypes: float64(2), int64(5), object(5)
memory usage: 83.7+ KB
"""


print(df.index)  # 输出: RangeIndex(start=0, stop=891, step=1)
print(df.columns)  # 默认第一列是列名字, 如果其它列是列明需要指定
""" 输出:
Index(['PassengerId', 'Survived', 'Pclass', 'Name', 'Sex', 'Age', 'SibSp',
       'Parch', 'Ticket', 'Fare', 'Cabin', 'Embarked'],
      dtype='object')
"""


print(df.dtypes)
""" 输出:
PassengerId      int64
Survived         int64
Pclass           int64
Name            object
Sex             object
Age            float64
SibSp            int64
Parch            int64
Ticket          object
Fare           float64
Cabin           object
Embarked        object
dtype: object
"""


print(df.values)
""" 输出:
[[1 0 3 ... 7.25 nan 'S']
 [2 1 1 ... 71.2833 'C85' 'C']
 [3 1 3 ... 7.925 nan 'S']
 ...
 [889 0 3 ... 23.45 nan 'S']
 [890 1 1 ... 30.0 'C148' 'C']
 [891 0 3 ... 7.75 nan 'Q']]
"""
```

## 创建自己的dataframe结构
### 创建dataframe结构
```
import numpy as np
import pandas as pd

// 数据为空的需要设置成 np.nan
data = {
    "country": ["aaa", "bbb", "ccc"],
    "population": [10, 12, np.nan],
    "name": ["Shanghai", "Beijing", "Guangzhou"],
    "location": [np.nan, "North", "Sourth"]
}
df_data = pd.DataFrame(data)
print(df_data)
// 输出:
  country  population       name location
0     aaa        10.0   Shanghai      NaN
1     bbb        12.0    Beijing    North
2     ccc         NaN  Guangzhou   Sourth
```

### 取指定数据
```
df_data_age = df_data["population"]
print(df_data_age[:2])
// 输出
0    10
1    12

print(df_data_age.values[:2])
// 输出
[10 12]
```

### 指定自己的索引
```
df_data = df_data.set_index("name")
print(df_data.head())
// 输出
          country  population location
name                                  
Shanghai      aaa        10.0      NaN
Beijing       bbb        12.0    North
Guangzhou     ccc         NaN   Sourth

df_data_population = df_data["population"]
print(df_data_population["Beijing"])
// 输出
12.0
```

### 获取数据基本统计特性
```
// 只能获取数值类型的数据的基本统计结果, 而对于其它列数据不能获取基本统计特性.
print(df_data.describe())
// 输出
       population
count    2.000000		// 表示population这一列有2个有效数据
mean    11.000000		// 均值
std      1.414214		// 标准差
min     10.000000		// 最小值
25%     10.500000		// 四分之一位数值
50%     11.000000		// 二分之一位数值
75%     11.500000		// 四分之三位数值
max     12.000000		// 最大值
```





