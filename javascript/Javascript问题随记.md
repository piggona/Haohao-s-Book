# Javascript问题随记

> javascript的坑很多很多，除了系统性的学一些，也要辅助一些零零碎碎的知识。

## 1. `==`与`===`

> `===`叫做严格运算符，`==`叫做相等运算符

### 严格运算符的运算规则：

1. 不同类型值

   如果两个值的类型不同，直接返回false

2. 同一类的原始类型值

   同一类型的<font color=#FF0033>原始类型</font>的值比较时，值相同就返回true，值不同就返回false

   > 原始类型：数值，字符串，布尔值

3. 同一类的复合类型值

   两个<font color=#FF0033>复合类型</font>的数据进行比较时，不是比较它们的值是否相等，而是比较它们是否指向<font color=#FF0033>同一个对象</font>

   > 复合类型：对象、数组、函数

4. undefined和null

   undefined和null与自身严格相等。

   ```javascript
   null === null //true
   undefined === undefined //true
   ```


### 相等运算符的运算规则：

> 相等运算符会先将数据进行类型转换，然后再用严格相等运算符比较

1. 原始类型的值：

   原始类型的数据会转换为数值类型进行比较。**字符串和布尔值都会转换成数值**

2. 对象与原始类型值比较：

   对象与原始类型的值进行比较时，对象会转换成原始类型的值。

3. undefined和null：

   undefined和null与其他类型的值比较时，结果都为false，它们互相比较时结果为true

4. 一些匪夷所思的例子：

   ```javascript
   '' == '0' // false
   0  == ''  // true
   0  == '0' // true
   
   false == 'false' // false
   false == '0'     // true
   
   false == undefined // false
   false == null      // false
   null  == undefined // true
   
   ' \t\r\n' == 0 //true
   ```
