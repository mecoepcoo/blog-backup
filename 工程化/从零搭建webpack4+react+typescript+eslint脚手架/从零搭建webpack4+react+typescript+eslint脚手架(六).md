---
title: 从零搭建webpack4+react+typescript+eslint脚手架(六)
date: 2019/10/6 16:12:00
updated: 2019/10/6 16:12:00
categories: 
- react
tags: 
- 工程化
- react
- webpack
- typescript
- eslint
---

本篇是前文的扩展延伸。

# 美化webpack输出信息
webpack在开发时的输出信息有一大堆，可能会干扰我们查看信息，以下提供一个美化、精简输出信息的建议。

精简以下开发服务器输出信息，修改`webpack.dev.js`：
```javascript
// ...webpack configs
stats: {
  colors: true,
  children: false,
  chunks: false,
  chunkModules: false,
  modules: false,
  builtAt: false,
  entrypoints: false,
  assets: false,
  version: false
}
```

美化一下打包输出，安装依赖：
```bash
$ npm i -D ora chalk
```

修改`config/build.js`：
```javascript
const ora = require('ora');
const chalk = require('chalk'); // 如果要改变输出信息的颜色，使用这个，本例没有用到
const webpack = require('webpack');
const webpackConfig = require('./webpack.prod');

const spinner = ora('webpack编译开始...\n').start();

webpack(webpackConfig, function (err, stats) {
  if (err) {
    spinner.fail('编译失败');
    console.log(err);
    return;
  }
  spinner.succeed('编译结束!\n');

  process.stdout.write(stats.toString({
    colors: true,
    modules: false,
    children: false,
    chunks: false,
    chunkModules: false
  }) + '\n\n');
});
```

分别运行打包和开发命令，控制台界面是不是清爽多了？

# 路由的配置
本段提供一个`react-router`的实践。

安装依赖：
```bash
$ npm i react-router-dom react-router-config @types/react-router-dom @types/react-router-config
$ npm i @loadable/component
```

新建`src/router.ts`：
```typescript
import loadable from '@loadable/component'; // 按需加载

export const basename = ''; // 如果访问路径有二级目录，则需要配置这个值，如首页地址为'http://tianzhen.tech/blog/home'，则这里配置为'/blog'

export const routes = [
  {
    path: '/',
    exact: true,
    component: loadable(() => import('@/pages/demo/HelloWorldDemo/HelloWorldDemoPage')), // 组件需要你自己准备
    name: 'home', // 自定义属性
    title: 'react-home' // 自定义属性
    // 这里可以扩展一些自定义的属性
  },
  {
    path: '/home',
    exact: true,
    component: loadable(() => import('@/pages/demo/HelloWorldDemo/HelloWorldDemoPage')),
    name: 'home',
    title: 'HelloWorld'
  },
  // 404 Not Found
  {
    path: '*',
    exact: true,
    component: loadable(() => import('@/pages/demo/404Page/404Page')),
    name: '404',
    title: '404'
  }
];
```

改造`index.tsc`，启用路由：
```typescript
import React from 'react';
import { BrowserRouter } from 'react-router-dom';
import { renderRoutes } from 'react-router-config';
import { routes, basename } from './router';
import '@/App.less';

const App: React.FC = () => {
  return <BrowserRouter basename={basename}>{renderRoutes(routes)}</BrowserRouter>;
};

export default App;
```

我们还可以利用路由为每个页面设置标题。

先写一个hook：
```typescript
import { useEffect } from 'react';

export function useDocTitle(title: string) {
  useEffect(() => {
    const originalTitle = document.title;
    document.title = title;
    return () => {
      document.title = originalTitle;
    };
  });
}
```

把hook应用在需要修改标题的组件中即可：
```typescript
import React from 'react';
import { useDocTitle } from '@/utils/hooks/useDocTitle';

import Logo from './react-logo.svg';
import './HelloWorldDemoPage.less';

const HelloWorldDemoPage: React.FC<Routes> = (routes) => {
  const { route } = routes; // 获取传入的路由配置
  useDocTitle(route.title); // 修改标题
  return <div className="App">hello, world</div>;
};

export default HelloWorldDemoPage;
```
