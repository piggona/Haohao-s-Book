# 前端编码记录：通用模块（HTML&CSS)

## 对于HTML模板需要配置HtmlWebPackPlugin()

> 配置流程：建立一个HtmlWebpackPlugin的实例->建立一个getHtmlConfig的配置实例->在plugin中new一个HtmlWebpackPlugin对象，使用getHtmlConfig配置

```javascript
var HtmlWebpackPlugin = require('html-webpack-plugin');
var getHtmlConfig = function(name){
    return {
        //模板路径
        template : './src/view/' + name + '.html',
        //打包文件路径
        filename : 'view/' + name + '.html',
        inject : true,
        hash : true,
        //打包的html中引入的js模块名
        chunks : ['common',name]
    }
};
var config ={
    plugins : [
        //...
        // html模板的处理
        new HtmlWebpackPlugin(getHtmlConfig('index')),
        new HtmlWebpackPlugin(getHtmlConfig('login'))
    ]
};
module.exports = config;
```

## loader配置完成之后就可以开始html的制作：包含模板页面和普通页面

> 普通页面调用模板页面使用以下代码：

```javascript
<%= require('html-loader!./layout/head-common.html') %>
```

## 之后开始配置css,首先要配置css,图片及字体的loader

### 1. css的loader配置：

> 我们要实现的css需要能在打包时提取为单独的文件而不是嵌入在html或js中，所以我们要使用ExtractTextPlugin

步骤为 实例化ExtractTextPlugin->配置对应的loader->配置css单独打包插件(plugins)

### 2. 图片及字体配置，主要在安装loader以及loaders的配置方面

安装url-loader(处理图片加载)

```bash
cnpm install url-loader --save-dev
```

安装字体库font-awesome

```bash
cnpm install font-awesome --save-dev
```

之后就可以配置webpack.config.js了

```javascript
var ExtractTextPlugin = require("extract-text-webpack-plugin");
var config = {
    module : {
        loaders: [
            { test: /\.css$/, loader: ExtractTextPlugin.extract("style-loader","css-loader") },
            { test: /\.(png|jpg|woff|svg|eot|ttf|gif|jpeg)\??.*$/, loader: "url-loader?limit=100000&name=resource/[name].[ext]" },
            { test: /\.(woff|svg|eot|ttf)\??.*$/, loader: 'style-loader!css-loader'},
        ],
    },
    plugins : [
        //独立通用模块
        new webpack.optimize.CommonsChunkPlugin({
            name : 'common',
            filename:'js/base.js'
        }),
        // 把css单独打包到文件中
        new ExtractTextPlugin("css/[name].css"),

        // html模板的处理
        new HtmlWebpackPlugin(getHtmlConfig('index')),
        new HtmlWebpackPlugin(getHtmlConfig('login'))
    ]
};
module.exports = config;
```

