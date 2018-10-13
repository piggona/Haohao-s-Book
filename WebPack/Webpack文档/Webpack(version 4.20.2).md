# Webpack(version 4.20.2)简略认识

> webpack本质上是一个现代的Javascript应用程序的静态模块打包器(module bundler)。当Webpack处理应用程序时，<font color=#FF0033>它会递归的梳理模块之间的依赖关系，构建一个依赖关系图(dependency graph)。</font>
>
> Webpack 4.0.0开始，可以不必要引入配置文件:`webpack.config.js`(使用高级功能还是需要配置，但为工程安装webpack的时候就默认不生成`webpack.config.js`了)

## 1. 核心概念

- ### 入口(entry)

  **入口起点(entry point)**指定了webpack使用哪个模块作为依赖关系图的起点。

  我们可以通过设置webpack.config.js中的`entry`属性来指定一个入口起点。默认值为<font color=#FF0033>`./src`</font>

  **webpack.config.js:**

  ```javascript
  module.exports = {
    entry: './path/to/my/entry/file.js'
  };
  ```

- ### 输出(output)

  **输出(output)**指定了webpack在1. 哪里输出它创建的bundles。2. 如何命名这些文件

  我们通过设置webpack.config.js中的`output`属性来指定输出。默认的输出路径为<font color=#FF0033>`./src`</font>

  **webpack.config.js:**

  ```javascript
  const path = require('path');
  
  module.exports = {
    entry: './path/to/my/entry/file.js',
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: 'my-first-webpack.bundle.js'
    }
  };
  ```

  ==*其中在webpack.config.js中及其它模块中直接require的模块是nodejs的核心模块*==

- ### loader

  **loader**让webpack处理*非JavaScript文件*（webpack本身只理解Javascript），loader工作的方式是将<font color=#FF0033>所有类型的文件转换为webpack能够处理的有效**模块**</font>

  配置loader主要有两个目标：

  1. `test`属性，标识应该被对应loader转换的某个，某类或某些文件。
  2. `use`属性，表示进行转换时，应该使用哪个loader

  ```javascript
  const path = require('path');
  
  const config = {
    output: {
      filename: 'my-first-webpack.bundle.js'
    },
    module: {
      rules: [
        { test: /\.txt$/, use: 'raw-loader' }
      ]
    }
  };
  
  module.exports = config;
  ```

- ### 插件(plugins)

  **loader**用于转换某些类型的模块，插件则用于执行范围更广的任务。插件的可以解决的问题：<font color=#FF0033>打包优化与压缩，重新定义环境中的变量，etc.</font>

  使用插件时进行的配置：

  1. 在webpack.config.js中require()指定插件
  2. 将require到的插件new(创建不同实例)在plugins数组中
  3. 多数插件可以通过选项(option)来自定义使用方式

  **webpack.config.js**

  ```javascript
  const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
  const webpack = require('webpack'); // 用于访问内置插件
  
  const config = {
    module: {
      rules: [
        { test: /\.txt$/, use: 'raw-loader' }
      ]
    },
    plugins: [
      new HtmlWebpackPlugin({template: './src/index.html'})
    ]
  };
  
  module.exports = config;
  ```

- ### 模式(mode)

  通过选择`development`或`production`之中的一个，来设置mode参数，你可以启用相应模式下的webpack内置的优化：

  ```javascript
  module.exports = {
    mode: 'production'
  };
  ```


