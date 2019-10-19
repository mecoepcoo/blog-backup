---
title: 从零搭建webpack4+react+typescript+eslint脚手架(四)
date: 2019/10/4 13:05:00
updated: 2019/10/4 13:05:00
categories: 
- react
tags: 
- 工程化
- react
- webpack
- typescript
- eslint
---

# 配置webpack开发服务器
打包用的配置基本完成了，现在我们来配置一下开发环境。

首先处理通用配置`config.js`：
```javascript
module.exports = {
  // ...configs
  devServer: {
    port: 8080,
    host: 'localhost',
    contentBase: path.join(__dirname, '../public'),
    watchContentBase: true,
    publicPath: '/',
    compress: true,
    historyApiFallback: true,
    hot: true,
    clientLogLevel: 'error',
    open: true,
    overlay: false,
    quiet: false,
    noInfo: false,
    watchOptions: {
      ignored: /node_modules/
    },
    proxy: {}
  }
};
```

然后增加开发配置：
```javascript
// webpack.dev.js
const path = require('path');
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');
const config = require('./config');

module.exports = merge.smart(baseWebpackConfig, {
  mode: 'development',
  output: {
    filename: 'js/[name].[hash:8].js',
    publicPath: config.publicPath
  },
  module: {
    rules: [
      {
        oneOf: []
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: config.indexPath,
      minify: {
        html5: true
      },
      hash: false
    }),
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    ...config.devServer
  }
});
```

在`package.json`中增加开发环境运行命令：
```json
"scripts": {
  "dev": "cross-env NODE_ENV=development webpack-dev-server --config ./config/webpack.dev.js"
}
```

运行`npm run dev`看看效果吧。

# 自定义多环境
一般来说，我们在开发应用的时候会面临多个环境差异的问题，例如，我们有：

- 一个开发环境，提交代码即可立刻看到效果，它的接口地址可能是`http://dev-api.tianzhen.tech`
- 一个测试环境，它需要保持一定程度的稳定性，每隔一小时发布一次新版本，接口地址可能是：`https://t1-api.tianzhen.tech`
- 预发布环境，它与生产环境共享持久化数据，在这个环境做最后一次检查，等待发布
- 生产环境，他需要保持高度稳定，一周发布一个版本，接口地址可能是：`https://api-tianzhen.tech`

四套环境，不同的接口地址，不同的访问地址，可能还涉及到不同的微信、支付宝鉴权。

许多人采用的方案是这样的，写几个不同的配置文件，切换环境时修改引入的配置，但是这样做经常会忘记切环境导致生产事故。这里提供一套自动多环境的配置方案。

依然先安装依赖：
```bash
$ npm i -D dotenv dotenv-expand # 从配置文件中读取并注入环境变量
$ npm i -D interpolate-html-plugin # 向模板注入环境变量
```

在根目录下新建几个环境配置文件：`.env`，`.env.dev`，`.env.prod`，文件名的格式是固定的，符合 `.env[.name][.local]`即可，同名的配置会按照优先级覆盖或自动合并，例如环境名称是`dev`，那么优先级就是`.env.dev.local`，`.env.dev`，`.env.local`，`.env`，高优先级覆盖低优先级。

我们随意编写一个环境变量配置：
```
// .env.dev
// 变量名要以 REACT_APP_ 开头
REACT_APP_ENV='dev'
REACT_APP_API_ROOT='http://dev-api.tianzhen.tech'
```

在`config`目录下新建一个`env.js`文件，用这个脚本来读取环境变量配置，用于以后注入到react项目中：
```javascript
const fs = require('fs');
const path = require('path');
const argv = require('yargs').argv;
const dotenv = require('dotenv');
const dotenvExpand = require('dotenv-expand');

const env = argv.env || 'production';
const ENV_FILE_PATH = path.resolve(__dirname, '../.env');

let dotenvFiles = [
  `${ENV_FILE_PATH}.${env}.local`,
  `${ENV_FILE_PATH}.${env}`,
  env !== 'test' && `${ENV_FILE_PATH}.local`,
  ENV_FILE_PATH
].filter(Boolean);

dotenvFiles.forEach((dotenvFile) => {
  if (fs.existsSync(dotenvFile)) {
    dotenvExpand(dotenv.config({
      path: dotenvFile
    }));
  }
});

const REACT_APP = /^REACT_APP_/i;

function getClientEnvironment(publicUrl) {
  publicUrl = process.env.NODE_ENV === 'production' ? publicUrl.slice(0, -1) : '';
  const raw = Object.keys(process.env)
    .filter(key => REACT_APP.test(key))
    .reduce(
      (env, key) => {
        env[key] = process.env[key];
        return env;
      },
      {
        NODE_ENV: process.env.NODE_ENV || 'production', // webpack在production模式下会自动启用一些配置
        APP_ENV: env,
        PUBLIC_URL: publicUrl
      }
    );
  
  const stringified = {};
  Object.keys(raw).forEach((key, index) => {
    stringified['process.env.' + key] = JSON.stringify(raw[key]);
  });

  return { raw, stringified };
}

module.exports = getClientEnvironment;
```

修改webpack配置，向react应用和index.html注入环境变量
```javascript
// webpack.base.js
const InterpolateHtmlPlugin = require('interpolate-html-plugin');
const getClientEnvironment = require('./env');

const env = getClientEnvironment(config.publicPath);

plugins: [
  new HtmlWebpackPlugin(),
  // 注意：注入插件一定要在HtmlWebpackPlugin之后使用
  // 在html模板中能够使用环境变量
  // <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
  new InterpolateHtmlPlugin(env.raw),
  // 在js代码中能够使用环境变量(demo: process.env.REACT_APP_ENV === 'dev')
  new webpack.DefinePlugin(env.stringified),
]
```

配置都做好了，如何让打包命令知道当前用的哪个环境呢，我们修改一下打包命令，加上`env`参数：
```json
"scripts": {
  "dev": "cross-env NODE_ENV=development webpack-dev-server --config ./config/webpack.dev.js --env=dev",
  "build:prod": "cross-env NODE_ENV=production node config/build.js --env=prod --report",
  "build:t1": "cross-env NODE_ENV=production node config/build.js --env=t1 --report",
  "build:dev": "cross-env NODE_ENV=production node config/build.js --env=dev --report"
}
```

把同样的配置，分别配置到`webpack.prod.js`和`webpack.dev.js`中，然后运行对应打包命令，就可以看到项目中成功注入了环境变量。例如，想要使用`.env.dev`中的变量，则打包命令中增加参数`--env=dev`即可，配置将由`.env.dev.local`，`.env.dev`，`.env.local`，`.env`合并覆盖生成。

> webpack根据`NODE_ENV`的值来自动选择`production`或`development`模式编译，因此，如果没有必须要求，尽量不要以`NODE_ENV`的值做为打包环境依据，否则就要自行处理更复杂的webpack配置。

# preload，prefetch
preload和prefetch是一组能够预读资源，优化用户体验的工具，这里给出一个在首页预读字体和图片的例子，来演示它们结合webpack的使用方法，详见[文档](https://github.com/GoogleChromeLabs/preload-webpack-plugin)。

安装依赖：
```bash
$ npm i -D preload-webpack-plugin
```

修改`webpack.prod.js`：
```javascript
const PreloadWebpackPlugin = require('preload-webpack-plugin')

plugins: [
  new PreloadWebpackPlugin({
    rel: 'preload',
    as(entry) {
      if (/\.css$/.test(entry)) return 'style';
      if (/\.woff$/.test(entry)) return 'font';
      if (/\.png$/.test(entry)) return 'image';
      return 'script';
    },
    include: ['app']
    // include:'allChunks'
  }),
]
```

# 配置按需加载
配置按需加载，可以将每个页面或组件拆成独立的包，减小首页加载内容的体积，是很好的优化策略。

安装依赖：
```bash
npm i -D @babel/plugin-syntax-dynamic-import
```

修改`webpack.base.js`
```javascript
{
  test: /\.(j|t)sx?$/,
  include: APP_PATH,
  use: [
    {
      loader: 'babel-loader',
      options: {
        plugins: [
          '@babel/plugin-syntax-dynamic-import', // 这是新加入的项
          ['@babel/plugin-proposal-class-properties', { 'loose': true }]
        ],
        cacheDirectory: true
      }
    }
  ]
}
```

配置完后，就可以用`import`的方式载入组件了：
```javascript
// demo
const HelloWorldPage = import('@/pages/demo/HelloWorldDemo/HelloWorldDemoPage');
```

至此，脚手架已经基本可以使用，并且完成了一部分优化。接下来的文章内容主要是围绕开发体验和团队规范展开的，还会涉及到一个比较优秀的react路由实践。
