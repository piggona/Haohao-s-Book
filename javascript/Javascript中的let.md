# Javascript中的let

[参考](https://www.jianshu.com/p/4e9cd99ecbf5)

## let相对于var的特性：

- #### let拥有块级作用域：

  let声明的变量的作用域只在它外层的块中（即`{}`包含的范围内）而var作用域是它外层的函数。

- #### let声明的全局变量不是全局对象(window)的直接属性

  let声明的变量无法通过`window.`方式访问

- #### 形如for (let x...)的循环在每次迭代时都为x创建新的绑定。

- #### let不能在外层重复定义