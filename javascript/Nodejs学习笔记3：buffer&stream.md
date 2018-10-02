# Nodejs学习笔记3：buffer&stream

## 1. Nodejs的Buffer

> 在javascript中，一般只提供字符串类型的数据，如果需要使用二进制数据，如TCP传输的数据，就需要使用Buffer这个类作为缓冲区来暂存传来的数据。

Buffer类似于数组的类型，是一个缓冲区可以写入读出建立。

### 1. Buffer与字符转换

Buffer可以提供编码转换的功能。

```javascript
const buf = new Buffer("hello haohao","ascii");
console.log(buf.toString('hex'));
console.log(buf.toString('base64'));
```

### 2. Buffer实例的创建

- 分配一定大小的初始化空间：Buffer.alloc(*分配空间的大小,分配空间的初始化值,空间的编码方式*)
- 分配没有初始化的空间：Buffer.allocUnsafe(size)
- 通过array建立一个Buffer实例：Buffer.from(array)
- 复制一个传入的Buffer(建立一个相同的Buffer)：Buffer.from(buf)
- 通过string与给定的编码建立一个新的Buffer实例：Buffer.from(string,encoding)

### 3. 写入Buffer

```javascript
buf.write(string[, offset[, length]][, encoding])
```

- string : 要写入的string
- offset : 缓冲区开始写入的索引值
- length：可以写入的字节数
- encoding : 编码，默认是utf8

写入过程主要是将string按encoding编码，之后写入buffer到offset规定的索引。如果超过length规定的写入字节数，那么后面的字节就不会被写入

### 4. 读出Buffer内容

```javascript
buf.toString([encoding[, start[, end]]])
```

- encoding : 使用的编码，默认为"utf-8"
- start : 开始读取的索引位置
- end : 结束位置，默认为缓冲区的末尾

```javascript
buf = Buffer.alloc(26);
for (var i = 0 ; i < 26 ; i++) {
  buf[i] = i + 97;
}

console.log( buf.toString('ascii'));       // 输出: abcdefghijklmnopqrstuvwxyz
console.log( buf.toString('ascii',0,5));   // 输出: abcde
console.log( buf.toString('utf8',0,5));    // 输出: abcde
console.log( buf.toString(undefined,0,5)); // 使用 'utf8' 编码, 并输出: abcde
```



### 5. 转Buffer内容为json

```javascript
buf.toJSON()
```

> 在字符串化一个Buffer实例时，JSON.stringify()会隐式调用toJSON()

```javascript
const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);

// 输出: {"type":"Buffer","data":[1,2,3,4,5]}
console.log(json);

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value.data) :
    value;
});

// 输出: <Buffer 01 02 03 04 05>
console.log(copy);
```

### 6. [其它缓冲区操作](http://www.runoob.com/nodejs/nodejs-buffer.html)

## 2. Nodejs Stream流

Stream是一个抽象接口，有四种流类型：

- Readable:可读操作
- Writable:可写操作
- Duplex:可读可写操作
- Transform:操作已经被写入的数据，并读出结果

**所有的Stream对象都是EventEmitter的实例：**

- data:当有数据可以读时触发
- end:没有更多数据可读时触发
- error:当接收和写入过程中出现错误时触发
- finish:所有数据都已经写入底层系统时触发

### 使用流可以：

- 读出流数据

- 写入流

- 从输出流到输入流

- 链式流（流操作链）

  [详情](http://www.runoob.com/nodejs/nodejs-stream.html)

