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
