# 数据预处理(Data Pre-processing)

> 数据预处理的过程对数据挖掘的过程具有重要的意义，本节主要讲解以下内容：
>
> - 数据类型
>   - 用于描述数据的类型存在差别(如定量&定性)
>   - 某些数据集有其特定的属性（特性），对挖掘结果至关重要
> - 数据的质量
>   - 噪声&离群点
>   - 缺失值
>   - 冗余数据（不一致或重复）
>   - 数据对我们关注的问题没有明显的影响，或两种数据对于结果的影响是一样的（非常接近）
> - 使数据适合挖掘的预处理步骤
>   - 为了方便数据挖掘进行一些处理（转换数据类型，清洗数据集）
> - 根据数据联系分析数据
>   - 找到数据对象之间的联系，之后使用这些联系而不是使用数据对象本身进行分析

## 1. 数据本身

### 属性与属性值（Attribute&Attribute Values）

> 属性与属性值是不同的：
>
> - 一个属性可能有多个属性值（如身高就有多个属性值依赖尺度值得不同）
> - 一个属性值也可能对应多个属性（如身份证号对应的可能是身份证号也可以只是串数字）

#### 测量标度（Measurement Scale）：

测量标度是将数值或符号与对象的属性相关联的规则（函数）

#### 属性的类型（Types of Attributes）：

1. ##### 属性的类型：

   - Nominal(标称)：
     相当于enum类型，可以列举的属性，没有顺序

   - Ordinal(序数)：
     **有序**的属性

   - Interval(区间值)：

     >  Example:calendar dates,tempetatures

   - Ratio(标度):

     有一个之前的数据比较而生成的标度

   ![image-20181023100727380](/Users/haohao/Documents/Haohao's-Book/数据仓储与数据挖掘/assets/image-20181023100727380.png)

2. ##### 属性类型的特性(Properties of Attibute Values)

   > 包含的特性有：
   >
   > - DIstinctness:相异性（等于|不等于）
   > - Order:有序性（<,>)
   > - Addition：加法（+，-）
   > - Multiplication:乘法（*，/）

   | 属性类型           | 特性                        |
   | ------------------ | --------------------------- |
   | Nominal attribute  | distinctness                |
   | Ordinal attribute  | distinctness&order          |
   | Interval attribute | distinctness&order&addition |
   | Ratio attribute    | all 4 properties            |

3. ##### 离散和连续的点(Discrete and Continuous)

### 数据集的类型(Types of data sets)

- #### 记录数据(Record)

  - Data Matrix
  - Document Data
  - Transaction Data

- #### 基于图形的数据(Graph)

  - World Wide Web
  - Molecular Data（分子结构）

- #### 有序的数据(Ordered)

  - Spatial Data（空间数据->坐标系内的）
  - Temporal Data（时间数据）
  - Sequential Data（顺序数据）
  - Genetic Sequence Data（基因序列数据）

#### 1. <font color=#FF0033>结构化数据</font>的重要特性

#### 2. Record Data

## 2. 数据预处理(Data Quality)