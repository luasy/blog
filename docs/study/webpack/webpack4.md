---

---

# Webpack 4 的使用学习

> 本文主要参考官方文档和自己想法写的。

学习webpack，肯定都知道它是一个帮你把代码进行一系列处理`(转化，压缩，合并等)`, 从而得到真正要发布的代码。
那么，我们由此大致可以知道，有如下几个配置。

1. Entry(入口): 从哪个文件开发打包.
2. Output(出口): 产生打包的文件打包到哪里.
3. Loader(加载器): 对源代码对转换。例如 `scss => css`、`ts => js`等。
4. Plugin(插件): 我认为可以理解为 `loader` 的一个`爸爸`, 它可以完成 `loader` 所不能完成的事情。因此，有某种意义上来说，`loader` 是 `plugin` 的一个精简版。插件有着众多的钩子函数，可以在各个特定的时间去触发，从而指定特定任务。

```js
const path = require('path')

module.exports = {
  entry: {},
  output: {},
  module: {
    loaders: [],
  },
  plugins: [],
}
```

## 一、工作原理

### 1.1 基本配置

webpack的工作原理先了解，便于理解以后学习局部的点。

