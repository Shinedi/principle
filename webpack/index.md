#### webpack考点梳理
---
1. 知识点
* 基本配置
* 高级配置
* 优化打包效率
* 优化产出代码
* 构建流程概述
* babel

2. 面试题
* 前端代码为何要进行构建和打包？
* module chunk bundle分别是什么意思,有何区别
* loader和plugin区别
* webpack如何实现懒加载
* webpack性能优化
* babel-runtime和babel-ployfill的区别

3. webpack基本配置
* 拆分配置和merge
* 启动本地server(可设置端口、代理)
* 处理ES6(babel-loader,配置babelrc(presets,plugins))
* 处理样式(loader执行顺序： 从后往前)
* 处理图片（线上base64，限制大小）
* 模块化
```
建common、dev、prod三个配置js,可以合并配置，例如合并common和dev
```

4. webpack高级配置
* 多入口配置
entry中配置多个入口，output中配置输出：[name].[hash].js,plugins里面配置多个htmlWebpackPlugin,里面要指定chunks,不然每个html都会引入所有的js
* 抽离css文件 - 重点
线上css文件的配置不能放在style中(style-loader是把css放在style标签中），安装`miniCssExtractPlugin`
```
{
    test: /\.css$/,
    loader: [
        MiniCssExtractPlugin.loader, // 注意：不再用style-loader
        'css-loader',
        'postcss-loader'
    ]
}
plugins: [
    // 抽离css文件
    new MiniCssExtractPlugin({
        filename: 'css/main.[conntentHash:8].css'
    })
],
optimizationn: [
    // 压缩
    minimizer: [new TerserJSPlugin({})]
]
```
* 抽离公共代码 - 重点
1. 第三方代码抽离
2. 公共引用代码抽离
```
optimization: [
    splitChunks: [
        chunks: 'all',
        cacheGroups: {
            // 第三方模块
            vendor: {
                name: 'vendor', // chunk名称
                priority: 1, //权限更高，优先抽离
                test: /node_modules/,
                minSize: 0, // 大小限制
                minChunks: 1, // 最少复用过几次
            },
            // 公共模块
            vendor: {
                name: 'common', // chunk名称
                priority: 1, //权限更高，优先抽离
                test: /node_modules/,
                minSize: 0, // 大小限制
                minChunks: 2, // 最少复用过几次
            }
        }
    ]
]
```
* 懒加载(文件过大) - 重点
```
setTimeout(import('./dy-k.js').then(res => {
    console.log(res.default.data)
}), 1000)
```
*产生chunk的地方:entry,分割代码，懒加载*

* 处理jsx和vue
安装@babel/preset-react,设置babelrc
安装vue-loader,配置在loader


5. module chunk bundle区别
module - 各个源码文件，webpack中一切皆模块(css,js，图片)
chunk - 多模块合并而成,通过入口文件做一些文件的分析,分析出很多集合 例如entry splitchunks import
bundle - 最终的输出文件

6. webpack性能优化
* 优化打包构建速度-开发体验和效率
    1) 优化babel-loader: 开启缓存cacheDirectory(es6代码没改，就可以加快速度)，include明确范围
    2) ignorePlugin: 避免引入无用模块
        a. 分析： 引入一个语言库,会默认引入所有语言的js代码，代码过大
        config.js
        ```
            plugins: [
                // 忽略moment下的/local目录
                new webpack.IgnorePlugin(/\.\/local/, /moment/)
            ]
        ```
        js文件
        ```
        import moment from 'moment'
        import 'moment/local/zh-cn'
        ```
    3) noparse
    ```
    module.exports = {
        module: {
            // 忽略对进行过模块化处理的js
            noParse: [react.js]
        }
    }
    ```
    区别
    ignorePlugin直接不引入,代码中没有,需要什么自己引
    noParse引入，不打包
    4) happyPack: 开启多进程打包
        js 是单线程,nodejs本身是单线程运行，开启多进程打包
        提高构建速度
        ```
        // 安装happypack
        const happyPack = require('happypack')
        module.exports = smart(webpackCommon, {
            ...
            module: {
                rules: [{
                    test: /js/
                    // 把对.js文件的处理交给id为babel的happypack实例
                    use: ['happypack/loader?id=babel']
                }]
            },
            plugins: [
                new HappyPack({
                    id: 'babel',

                })
            ]
        })
        ```
    5) parallelUglifyPlugin多进程压缩js（生产环境）
        a. webpack内置uglify工具压缩js
        b. js是单线程,开启多进程压缩更快
        ```
        // 还是使用uglify，只不过开启了多进程
        new 
        ```
        关于开启多进程
        * 项目较大，打包较慢,开启多进程能提高速度
        * 项目较小，打包很快,开启多进程会降低速度(进程开销)
        * 按需使用
    6) 自动刷新（开发环境）
        引入devServer就会有这个功能
        整个网页全部刷新，速度较慢,状态会丢失
    7) 热更新（开发环境）
        新代码生效，网页不刷新,状态丢失
        引入webpack插件hotmodule
        entry: webpack
        plugins设置new hotMudule
        devserver设置hot为true
        增加开启热更新的逻辑
    8) dllPlugin： 动态链接库插件(开发环境)
        前端框架如vue,体积大，构建慢
        较稳定，不常升级版本
        同一个版本只构建一次即可,不用每次都重新构建
        思路
        ```
        webpack已经内置dllplugin支持
        包含2个插件
        dllPlugin-打包出dll文件
        dllRefernceplugin-使用这个文件

        ``` 

7. 优化产出代码
    优点
    ---
    体积更小
    合理分包，不重复加载
    速度更快，内存使用更少
    措施
    ---
    小图片base64
    bundle加hash:根据文件内容算出hash值
    懒加载
    提取公共代码:splitchunks
    IgnorePlugin: 体积更小
    使用cdn加速: output中配置cdn前缀publicPath,把打包出来的文件(js,css,img)放在cdn机器上
    使用production
    使用scope hosting
    ___
    使用procuction
    可以自动压缩代码
    vue react等会自动删除掉调试代码
    启动tree shakinng(使用es6module才能让commonjs生效，commonjs不行)
8. es6module和commonjs区别
es6module静态引入，编译时引入
commonjs动态引入，执行时引入
只有es6module才能静态分析,实现tree-shaking

9. scope hoisting
多个函数合并，代码体积更小
创建函数作用域更少
代码可读性更好

10. babel-polyfill(补丁)是什么？ 
core-js和regenerator
babel-polyfill是两者的集合
缺点
---
会污染全局变量

### 面试题
1. 前端为何要进行打包和构建
* 体积更小(tree-shaking,压缩，合并),加载更快
* 编译高级语言和语法(es6+)
* 兼容性和错误检查（）

统一高效的开发环境
统一的构建流程和产出标准
集成公司构建规范(提测、上线等)

2. module bundle, chunk区别
module - 各个源码文件，webpack中一切皆模块(css,js，图片)
chunk - 多模块合并而成,通过入口文件做一些文件的分析,分析出很多集合 例如entry splitchunks import
bundle - 最终的输出文件

3. loader和plugin的区别
loader模块转换器,如less->css
plugin扩展组件

4. 常见loader和plugin

5. babel和模块化区别
babel-js语法编译工具，不关心模块化
webpack-打包构建工具,是多个loader plugin的集合

6. 如何产出一个lib
liberay: 'lodash'

7. webpack如何实现懒加载
import

8. 为何proxy不能被ployfill? 
* 如class可以用function模拟
* 如Promise可以用callback模拟
* proxy的功能用Object.defineProperty无法模拟
