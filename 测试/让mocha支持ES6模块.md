---
title: 让mocha支持ES6模块
date: 2018/9/26 21:00:00
categories: 
- 测试
tags: 
- 测试
- 工程化
---

`mocha`是比较常用的node测试框架，但是只支持`commonjs`模块，要让`mocha`支持ES6模块，需要`babel`的帮助。

书写本文时用到的工具版本为：

- babel `v7`
- mocha `v6.2`

# 安装依赖
```bash
$ npm i -D @babel/cli @babel/core @babel/preset-env @babel/register
```

# babel配置
在`package.json`或`.babelrc`中添加配置：
```js
{
  "presets": ["@babel/preset-env"]
}
// "babel": {
//   "presets": [
//     "@babel/preset-env"
//   ]
// }
```

# 配置命令
最后配置运行命令，`babel/register`会绑定到node的`require`模块，代码运行时会实施转译，这样就可以支持ES6的模块语法了：
```json
"scripts": {
  "test": "mocha --require @babel/register test/*.js",
}
```
