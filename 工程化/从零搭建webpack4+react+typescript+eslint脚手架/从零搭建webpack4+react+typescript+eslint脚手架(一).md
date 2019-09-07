# 引言
> 项目github仓库地址： [https://github.com/mecoepcoo/ts-react-boilerplate](https://github.com/mecoepcoo/ts-react-boilerplate)
> 
这个系列的文章主要讲述如何从一个空目录建立**webpack+react+typescript+eslint**脚手架，书写此文时各主要工具的版本为：
- webpack `v4`
- react `v16.9`
- typescript `v3.5`
- babel `v7`
- eslint `v6.2`

本文涉及的内容大致包含：
- webpack的配置
- 对静态资源（图片，模板等）的处理
- 使react项目支持typescript，eslint，prettier等工具
- 优化webpack配置，减小代码的体积
- 支持不同的css预处理器（less，sass等）
- 一套好用的样式方案
- 使项目支持多个环境切换（开发，测试，预发布，生产等）
- 使用规则来自动约束代码规范
- 优化开发体验
- 一些优化项目性能的建议

> Why not [create-react-app](https://create-react-app.dev/)?
> 
> 笔者使用**CRA**新建项目时，觉得自定义程度不够。尝试过 `react-app-rewired + customize-cra` 的方案，还是觉得异常繁琐，而且会消耗额外的维护精力，**鲁迅说**：青年应当有朝气，敢作为，遂自行搭建一个 boilerplate 。

# 初始化目录
我们要从一个空目录开始，先新建这个目录，做一些必要的初始化工作：
```bash
$ mkdir my-react
$ cd my-react

$ git init
$ npm init
```

新建如下目录结构：
react-project
- config `打包配置`
- public `静态文件夹`
  - index.html
  - favicon.ico
- src `源码目录`

# 规范git提交
协作开发时，git提交的内容如果没有规范，就不好管理项目，我们用 [husky](https://github.com/typicode/husky) + [commitlint](https://github.com/conventional-changelog/commitlint) 来规范git提交。

我们先在根目录下建立 `.gitignore` 文件，忽略不需要要的文件。

然后安装工具：
```
$ npm i -D husky
$ npm i -D @commitlint/cli
```

> husky 会为 git 增加钩子，在 commit 时执行一系列操作，commitlint 可以检查 git message 是否符合规则。

在 `package.json` 中增加配置如下：
```json
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
},
```

在根目录新建文件 `.commitlintrc.js`，根据具体情况配置：
```javascript
module.exports = {
  parserPreset: {
    parserOpts: {
      headerPattern: /^(\w*)(?:\((.*)\))?:\s(.*)$/,
      headerCorrespondence: ['type', 'scope', 'subject']
    }
  },
  rules: {
    'type-empty': [2, 'never'],
    'type-case': [2, 'always', 'lower-case'],
    'subject-empty': [2, 'never'],
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'update', 'docs', 'style', 'refactor', 'test', 'chore', 'release', 'revert']
    ]
  }
}
```

这样即可完成配置，具体的使用方法参考 [commitlint文档](https://github.com/conventional-changelog/commitlint)

# React hello, world
安装react，写一个react hello, world

现在让主角 React 登场：
```bash
$ npm i react react-dom
```

新建一个 `hello, world` 结构，这里直接用ts书写：
```tsx
// src/index.tsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import './style.css';

ReactDOM.render(<App />, document.getElementById('root'));
```

```tsx
// src/App.tsx
import React from 'react';
import './app.css';

const App: React.FC = () => {
  return (<div>hello, world</div>);
};

export default App;
```

我们还需要一个html模板：
```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8" />
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
    <title>react-app</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```

完整的结构参考 [代码示例](https://github.com/mecoepcoo/ts-react-boilerplate)

# webpack的基本配置
安装webpack相关工具：
```bash
$ npm i -D webpack webpack-cli webpack-dev-server webpack-merge
```

在 config 目录下新建几个文件：`config.js`, `webpack.base.js`, `webpack.prod.js`, `webpack.dev.js`, `build.js`

先配置一些通用的：
```javascript
// config/config.js
const path = require('path');

module.exports = {
  assetsRoot: path.resolve(__dirname, '../dist'),
  assetsDirectory: 'static',
  publicPath: '/',
  indexPath: path.resolve(__dirname, '../public/index.html'),
};
```

```javascript
// config/webpack.base.js
const path = require('path');
const webpack = require('webpack');
const config = require('./config');

module.exports = {
  entry: {
    app: './src/index.tsx',
  },
  output: {
    filename: 'js/[name].bundle.js',
    path: config.assetsRoot,
    publicPath: config.publicPath
  },
  module: {
    rules: [
      {
        oneOf: []
      }
    ]
  },
  resolve: {
    extensions: ['.js', '.json', '.jsx', '.ts', '.tsx'] // 自动判断后缀名，引入时可以不带后缀
  },
  plugins: []
};
```

# babel和typescript，路径别名
接下来我们需要让webpack支持typescript，并且将代码转换为es5，这样才能在低版本的浏览器上运行。

依然是先安装工具：
```bash
$ npm i -D babel-loader @babel/core @babel/preset-env @babel/preset-react @babel/polyfill
$ npm i core-js@2 # babel的按需引入依赖
$ npm i -D @babel/plugin-proposal-class-properties # 能够在class中自动绑定this的指向
$ npm i -D typescript awesome-typescript-loader # 处理ts，主要就靠它
```

用了ts，就要有一个tsconfig配置，在根目录新建 `tsconfig.json`：
```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "typeRoots": [
      "src/types" // 指定 d.ts 文件的位置，根据具体情况修改
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react",
    "baseUrl": ".",
  },
  "include": [
    "src"
  ],
  "exclude": [
    "./node_modules"
  ]
}
```

来配一下webpack：
```javascript
// webpack.base.js
const APP_PATH = path.resolve(__dirname, '../src');

module: {
  rules: [
    {
      oneOf: [
        {
          test: /\.(j|t)sx?$/,
          include: APP_PATH,
          use: [
            {
              loader: 'babel-loader',
              options: {
                presets: [
                  '@babel/preset-react',  // jsx支持
                  ['@babel/preset-env', { useBuiltIns: 'usage', corejs: 2 }] // 按需使用polyfill
                ],
                plugins: [
                  ['@babel/plugin-proposal-class-properties', { 'loose': true }] // class中的箭头函数中的this指向组件
                ],
                cacheDirectory: true // 加快编译速度
              }
            },
            {
              loader: 'awesome-typescript-loader'
            }
          ]
        },
      ]
    }
  ]
}
```

为了以后开发时引入路径方便，我们加个路径别名的配置，需要改webpack配置和tsconfig两处：
```javascript
// webpack.base.js
resolve: {
  extensions: ['.js', '.json', '.jsx', '.ts', '.tsx'],
  alias: {
   '@': path.resolve(__dirname, '../src/') // 以 @ 表示src目录
  }
 },
```

```json
{
  "compilerOptions": {
    // ...
    "paths": {
      "@/*": ["src/*"]
    }
    // ...
  }
}
```

至此，我们完成了最最基本的webpack配置，但暂时还不能打包。
