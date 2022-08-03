---
title: webpack多入口打包环境搭建
date: 2022-07-27 23:23
---
创建一个新目录执行 `npm init -y`，生成 `package.json`。

在当前目录下执行以下命令安装依赖

> 本次项目中使用scss如有不同自行替换，html文件由[ejs模版](https://ejs.co/)生成，使用html-webpack-plugin生成html文件，防止打包存在缓存文件使用clean-webpack-plugin清除目录，使用webpack-merge合并webpack配置

```shell
npm i webpack webpack-cli webpack-dev-server sass sass-loader css-loader style-loader ejs-loader html-webpack-plugin clean-webpack-plugin webpack-merge --save-dev
```



在当前目录下新建entry文件夹，由于webpack是基于js文件为入口打包，所以在entry入口下新建对应的页面的入口js

![](https://image.liuyongzhi.cn/images/entry-dir.png)

在当前目录下新建build文件夹用来存放webpack打包配置

![](https://image.liuyongzhi.cn/images/webpack-dir.png)

[webpack](https://webpack.docschina.org/concepts/)配置这里不过多赘述，只需要修改entry即可

![](https://image.liuyongzhi.cn/images/entry-code.png)

在根目录下创建工程目录

```
|-- build
	  |-- webpack.config.js
	  |-- webpack.dev.js
	  |-- webpack.build.js
|-- entry
    |-- index.js
    |-- featrues.js
    |-- main.js
|-- src
	  |-- styles
	  		|-- featrues.scss
	  		|-- index.scss
	  |-- pages
	  		|-- featrues
	  				|-- index.ejs
	  		|-- index
	  			  |-- index.ejs
		|-- components
				|-- header.ejs
				|-- footer.ejs
```

配置webpack插件

> 此处可以手写函数扫描文件夹名称以达到循环生成模版

![](http://image.liuyongzhi.cn/images/webpack-plugin.png)

最后增加出口

![](http://image.liuyongzhi.cn/images/webpack-output.png)

中间的loader处理就不写了，文档有固定的用法，最后贴出webpack的基础配置，可以在此基础在进行丰富

```js
// webpack.config.js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin') 
module.exports = {
    entry: {
        index: './entry/index.js',
        featrues: './entry/featrues.js',
        main: './entry/main.js',
    },
    output: {
        filename: '[name].js',
        path: __dirname + '../dist',
    },
    resolve: {
        alias: {
            '@src': path.resolve(__dirname, '../src')
        }
    },
    optimization: {
        minimizer: [
            new OptimizeCssAssetsWebpackPlugin()
        ]
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            },
            {
                test: /\.scss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    'sass-loader'
                ]
            },
            {
                test: /\.ejs$/,
                use: [
                    {
                        loader: 'ejs-loader',
                        options: {
                            esModule: false,
                            variable: 'data',
                        },
                    },
                ],
            },
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: path.resolve(__dirname, `../src/pages/index/index.ejs`),
            filename: `index.html`,
            inject: false,
            hash: true
        }),
        new HtmlWebpackPlugin({
            template: path.resolve(__dirname, `../src/pages/featrues/index.ejs`),
            filename: `featrues.html`,
            inject: false,
            hash: true
        }),
    ]
}

```

```js
// webpack.build.js
const { merge } = require('webpack-merge')
const webpackConfig = require('./webpack.config')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
module.exports = merge(webpackConfig, {
    mode: "production",
    plugins: [
        new CleanWebpackPlugin()
    ]
})
```

```js
// webpack.dev.js
const { merge } = require('webpack-merge')
const path = require('path')
const webpackConfig = require('./webpack.config')
module.exports = merge(webpackConfig, {
    mode: "development",
    devServer: {
        port: 4000,
        host: '0.0.0.0',
        hot: true,
        open: true,
        static: {
            directory: path.resolve(__dirname, '../src/assets'),
            publicPath: '/'
        },
        compress: true,
        client: {
            logging: 'info',
            progress: true,
            overlay: {
                errors: true,
                warnings: false
            }
        }
    }
})
```

