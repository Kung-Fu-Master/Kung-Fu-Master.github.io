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





