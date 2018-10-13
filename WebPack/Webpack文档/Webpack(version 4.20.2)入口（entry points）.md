# Webpack(version 4.20.2)入口（entry points）

> 配置webpack需要打包的工程的入口文件，配置webpack.config.js->config中的entry属性来配置。

## 1. 单个入口写法：

用法：`entry: string|Array<string>`

##### Webpack.config.js

```javascript
const config = {
    entry: ['./path/to/my/entry/file.js',]
};
module.exports = config
```

如果entry属性传入的是多个文件路径（数组），那么就会创建**”多个主入口(multi-main entry)“**，就是将多个入口一起及其依赖一起注入，最后作为一个chunk输出

## 2. 对象语法

用法：`entry: {[entryChunkName: string]: string|Array<string>}`

##### Webpack.config.js

```javascript
const config = {
    entry: {
        app: './src/app.js',
        vendors: './src/vendors.js'
    }
};
```

> **”可扩展的webpack配置“**是指，可重用并且可以与其他配置组合使用。这是一种流行的技术，用于将关注点(concern)从环境(environment)、构建目标(build target)、运行时(runtime)中分离。然后使用专门的工具（如[webpack-merge](https://www.webpackjs.com/concepts/entry-points/))将他们合并，<font color=#FF003>模块化的进行分离开发</font>

## 3. 常见场景

- #### 分离应用程序(app)和 第三方库(vendor)入口

  > 在只有一个入口起点的单页面应用程序(single page application)中，<font color=#FF0033>使用CommonChunkPlugin从应用程序的javascript代码(叫做app bundle)中提取vendor引用(vendor reference)到vendor bundle</font>,并把引用vendor的部分替换为`__webpack_require__()`调用。如果应用程序bundle中没有vendor代码，那么可以在webpack中实现被称为[长效缓存](https://www.webpackjs.com/guides/caching)的通用模式

  ##### Webpack.config.js

  ```javascript
  const config = {
    entry: {
      app: './src/app.js',
      vendors: './src/vendors.js'
    }
  };
  ```

- #### 多页面应用程序

  > 多页面应用程序的痛点：每次请求服务器都需要传输一个新的HTML文档，页面需要重新加载资源

  ##### Webpack.config.js

  ```javascript
  const config = {
    entry: {
      pageOne: './src/pageOne/index.js',
      pageTwo: './src/pageTwo/index.js',
      pageThree: './src/pageThree/index.js'
    }
  };
  ```

  <font color=#FF0033>使用CommonChunk插件为每个页面间的应用程序共享代码创建bundle,多页应用能共享大量代码/模块</font>