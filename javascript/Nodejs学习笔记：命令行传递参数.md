# Nodejs学习笔记：命令行传递参数

[参考](http://www.ruanyifeng.com/blog/2015/05/command-line-with-node.html)

> 主要用于用nodejs开发命令行程序或提供外部程序可以调用的js可执行文件

## 1. 简单的使用命令行参数

变量process.argv代表命令行中输入的元素

 例：

对于一个简单的脚本

```javascript
// 位置:./hello.js
console.log('hello',process.argv[2]);
```

如果要传递一个参数则命令行：

```bash
> $ node hello.js tom
```

则结果返回:

```javascript
hello tom
```

<font color=#FF0033>其中`argv[2]`代表命令行中的第三个元素，即tom(第一个元素：node,第二个元素：hello.js)。</font>

## 2. 处理命令行参数（yargs模块）

yargs需要安装后才可以使用：

```javascript
npm install --save yargs
```

yargs中提供argv对象，来<font color=#FF0033>读取命令行参数</font>

```javascript
// 位置：./hello.js
var argv = require('yargs').argv;

console.log('hello',argv.name);
```

命令行输入：

```javascript
> $ node hello.js --name=tom

> $ node hello.js --name tom
```

可以使用alias方法，指定参数的别名：

```javascript
var argv = require('yargs').alias('n','name').argv;

console.log('hello',argv.n);
```

这样使用argv.n就能表示命令行中的参数argv.name。

使用`_`来表示忽略匹配：

```javascript
var argv = require('yargs').alias('n','name').argv;

console.log('hello',argv.n);
console.log(argv._);
```

## 3. yargs中关于命令行参数的其它配置

命令行选项：

- Demand:是否必选
- Deault:默认值
- Describe:提示

```javascript
var argv = require('yargs')
  .demand(['n'])
  .default({n: 'tom'})
  .describe({n: 'your name'})
  .argv;

console.log('hello ', argv.n);
```

也可以将一个参数的配置写到一个option中：

```javascript
var argv = require('yargs')
  .option('n', {
    alias : 'name',
    demand: true,
    default: 'tom',
    describe: 'your name',
    type: 'string'
  })
  .argv;

console.log('hello ', argv.n);
```

