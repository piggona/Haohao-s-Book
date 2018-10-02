# Nodejs学习笔记4：模块系统

## 1. 引用模块

```javascript
var hello = require('./hello');
hello.world();
```

使用require的方法引入模块。

## 2. 创建模块

### 1. 函数模块

```javascript
exports.world = function() {
  console.log('Hello World');
}
```

这样建立一个函数模块，在调用的时候：

```javascript
var hello = require('./hello');
hello.world();
```

就可以使用这个函数

### 2. 类模块

```javascript
function Hello() { 
    var name; 
    this.setName = function(thyName) { 
        name = thyName; 
    }; 
    this.sayHello = function() { 
        console.log('Hello ' + name); 
    }; 
}; 
module.exports = Hello;
```

声明类模块时指定类函数的名称，在export时导出的是Hello这个类函数。

使用这个模块时：

```javascript
var Hello = require('./hello'); 
hello = new Hello(); 
hello.setName('BYVoid'); 
hello.sayHello(); 
```

