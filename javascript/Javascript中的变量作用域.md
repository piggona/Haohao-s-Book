# Javascript中的变量作用域

[参考](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/0014344993159773a464f34e1724700a6d5dd9e235ceb7c000)

## 1. 简单的变量作用域：

javascript中的函数在查找变量时从自身函数的定义开始，从“内”向“外”查找：

1. 内部的函数可以访问外部函数定义的变量，反之则不可
2. 如果内部函数定义了与外部函数重名的变量，则内部函数的变量就会屏蔽外部函数的变量

```javascript
'use strict';

function foo() {
    var x = 1;
    function bar() {
        var x = 'A';
        console.log('x in bar() = ' + x); // 'A'
    }
    console.log('x in foo() = ' + x); // 1
    bar();
}

foo();

//output:
//x in foo() = 1
//x in bar() = A
```

如果bar()内部没有定义x变量，那么bar()里面调用x变量时就会到外面的函数中去寻找。

```javascript
'use strict';

function foo() {
    var x = 1;
    function bar() {
        console.log('x in bar() = ' + x); // 'A'
    }
    console.log('x in foo() = ' + x); // 1
    bar();
}

foo();

//output:
//x in foo() = 1
//x in bar() = 1
```

## 2. 变量提升

javascript会先扫描整个函数体的语句，将所有申明的变量提升到函数顶部。但是只是把<font color=#FF0033>**申明**</font>提升到顶部，而不会把<font color=#FF0033>**赋值**</font>语句提升到顶部。

例：

```javascript
'use strict';

function foo() {
    var x = 'Hello, ' + y;
    console.log(x);
    var y = 'Bob';
}

foo();
```

运行这段代码时javascript就会将代码变为如下的形式：

```javascript
function foo() {
    var y; // 提升变量y的申明，此时y为undefined
    var x = 'Hello, ' + y;
    console.log(x);
    y = 'Bob';
}
```

只提升了y变量的申明，赋值语句还是在下面。

为了避免这种代码混乱的情况，一般我们在函数头就把所有变量声明并初始化赋值。

## 3. 全局作用域(Window对象)

不在任何函数内定义的变量就有全局作用域，所有这种变量其实都是在一个window对象中，具有全局作用域的属性(var)/方法(function)就是被绑定到window的一个属性。

```javascript
'use strict';

window.alert('调用window.alert()');
// 把alert保存到另一个变量:
var old_alert = window.alert;
// 给alert赋一个新函数:
window.alert = function () {}
alert("无法显示")
```

## 4. 建立自己工程的名字空间

**为了保护工程中的全局变量不与window即全局作用域的变量发生冲突。一般会将本工程的全部全局变量和函数都绑定到一个全局变量中**

例：

```javascript
// 唯一的全局变量MYAPP:
var MYAPP = {};

// 其他变量:
MYAPP.name = 'myapp';
MYAPP.version = 1.0;

// 其他函数:
MYAPP.foo = function () {
    return 'foo';
};
```

## 5. 局部作用域（块级作用域）

javascript变量作用域实际上是函数的内部，在for循环等<font color=#FF0033>**语句块**</font>中无法定义具有局部作用域的变量：

```javascript
'use strict';

function foo() {
    for (var i=0; i<100; i++) {
        //
    }
    i += 100; // 仍然可以引用变量i
}
```

ES6中引入了新的关键字let,用let替代var可以声明一个块级作用域的变量：

```javascript
'use strict';

function foo() {
    var sum = 0;
    for (let i=0; i<100; i++) {
        sum += i;
    }
    // SyntaxError:
    i += 1;
}
```

#### **常量赋值**

要声明一个常量：

- 使用格式（使用大写字母）来声明一个变量，来约定常量（只能约定）

- 在ES6标准中引入了新的关键字const来定义常量，const与let都有块级作用域

  ```javascript
  'use strict';
  
  const PI = 3.14;
  PI = 3; // 某些浏览器不报错，但是无效果！
  PI; // 3.14
  ```


## 6. 解构赋值(ES6)：将一个数组/对象赋值给数组/对象

- 对数组类型

  > 将一个数组对象赋值给多个变量

  ```javascript
  'use strict';
  
  // 如果浏览器支持解构赋值就不会报错:
  var [x, y, z] = ['hello', 'JavaScript', 'ES6'];
  ```

  如果数组本身存在嵌套关系：

  ```javascript
  let [x, [y, z]] = ['hello', ['JavaScript', 'ES6']];
  x; // 'hello'
  y; // 'JavaScript'
  z; // 'ES6'
  ```

  解构时可以忽略一些元素：

  ```javascript
  let [, , z] = ['hello', 'JavaScript', 'ES6']; // 忽略前两个元素，只对z赋值第三个元素
  z; // 'ES6'
  ```

- 对对象类型

  > 取对象中的一些属性

  ```javascript
  var person = {
      name: '小明',
      age: 20,
      gender: 'male',
      passport: 'G-12345678',
      school: 'No.4 middle school',
      address: {
          city: 'Beijing',
          street: 'No.1 Road',
          zipcode: '100001'
      }
  };
  var {name, address: {city, zip}} = person;
  name; // '小明'
  city; // 'Beijing'
  zip; // undefined, 因为属性名是zipcode而不是zip
  
  // 注意: address不是变量，而是为了让city和zip获得嵌套的address对象的属性:
  address; // Uncaught ReferenceError: address is not defined
  ```

  如果使用的变量名和属性名不同，那么可以使用赋值语句(:)来实现：

  ```javascript
  var person = {
      name: '小明',
      age: 20,
      gender: 'male',
      passport: 'G-12345678',
      school: 'No.4 middle school'
  };
  
  // 把passport属性赋值给变量id:
  let {name, passport:id} = person;
  name; // '小明'
  id; // 'G-12345678'
  // 注意: passport不是变量，而是为了让变量id获得passport属性:
  passport; // Uncaught ReferenceError: passport is not defined
  ```

  解构赋值也可以使用默认值，避免不存在的属性报错：

  ```javascript
  var person = {
      name: '小明',
      age: 20,
      gender: 'male',
      passport: 'G-12345678'
  };
  
  // 如果person对象没有single属性，默认赋值为true:
  var {name, single=true} = person;
  name; // '小明'
  single; // true
  ```

  如果变量已经被声明，再次赋值时，正确的写法也会报语法错误：

  ```javascript
  / 声明变量:
  var x, y;
  // 解构赋值:
  {x, y} = { name: '小明', x: 100, y: 200};
  // 语法错误: Uncaught SyntaxError: Unexpected token =
  ```

  因为Javascript把{开头的语句当做一个块来处理，于是=不再合法，解决方法：

  ```javascript
  ({x, y} = { name: '小明', x: 100, y: 200});
  ```

## 7. 词法作用域（lexical scoping)&动态作用域

[参考](https://segmentfault.com/a/1190000008972987)

**javascript采用词法作用域**

> 词法作用域：函数的作用域在函数定义的时候就确定
>
> 动态作用域：函数的作用域在函数调用的时候才决定

例：

```javascript
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();
```

执行 foo 函数，先从 foo 函数内部查找是否有局部变量 value，如果没有，就根据书写的位置，查找上面一层的代码，也就是 value 等于 1，所以结果会打印 1。

假设JavaScript采用动态作用域，让我们分析下执行过程：

执行 foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。如果没有，就从调用函数的作用域，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。

前面我们已经说了，JavaScript采用的是静态作用域，所以这个例子的结果是 1。