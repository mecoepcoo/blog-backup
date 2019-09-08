# 完善webpack打包配置
有了webpack的基础配置，还不足以支持打生产环境能够使用的包，我们还需要增加一些配置。

首先，每次打包前最好能把上一次生成的文件删除，这里可以用[clean-webpack-plugin](https://github.com/johnagan/clean-webpack-plugin)插件实现：
```
$ npm i -D clean-webpack-plugin
```

然后修改webpack基础配置：
```javascript
// webpack.base.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  plugins: [
    new CleanWebpackPlugin(),
  ]
}
```

在生产环境，我们希望部署新版本后能够丢弃缓存，又希望保留没有被改动的文件的缓存，而在开发环境，我们希望完全不使用缓存，因此我们需要在当前配置的基础上，分别扩展生产和开发两套配置。

```javascript
// webpack.prod.js 生产环境打包配置
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = merge.smart(baseWebpackConfig, {
  mode: 'production',
  devtool: sourceMapsMode,
  output: {
    filename: 'js/[name].[contenthash:8].js', // contenthash：只有模块的内容改变，才会改变hash值
  },
  plugins: [
    new CleanWebpackPlugin(),
  ]
}
```

```javascript
// webpack.dev.js 开发环境的配置
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base');
const config = require('./config');

module.exports = merge.smart(baseWebpackConfig, {
  mode: 'development',
  output: {
    filename: 'js/[name].[hash:8].js',
    publicPath: config.publicPath // 这里可以省略
  },
  module: {
    rules: [
      {
        oneOf: []
      }
    ]
  },
}
```

接下来我们编辑`build.js`，让打包程序真正能够运行起来：
```javascript
// config/build.js
const webpack = require('webpack');
const webpackConfig = require('./webpack.prod');

webpack(webpackConfig, function (err, stats) {});
```

安装工具并添加启动命令：
```bash
$ npm i -D cross-env
```

```json
// package.json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --config ./config/webpack.dev.js",
    "build": "cross-env NODE_ENV=production node config/build.js"
  }
}
```

然后运行打包命令，就能看到新生成的dist目录中有已经打包好的文件了：
```bash
$ npm run build
```

# 打包分析工具
包是打出来了，但是打包好的文件构成是什么样呢，有没有按照我们的需要正确打包呢，我们需要一个分析工具来帮助判断，这就是[webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)。

```bash
$ npm i -D webpack-bundle-analyzer
```

我们希望根据打包的命令参数，在打包时自动生成或不生成分析报告。

```javascript
// webpack.base.js
const argv = require('yargs').argv;
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
const merge = require('webpack-merge');

const bundleAnalyzerReport = argv.report; // 根据命令参数是否含有 'report' 来决定是否生成报告
// 这个配置将合并到最后的配置中
const webpackConfig = {
  plugins: []
};
if (bundleAnalyzerReport) {
  webpackConfig.plugins.push(new BundleAnalyzerPlugin({
    analyzerMode: 'static',
    openAnalyzer: false,
    reportFilename: path.join(config.assetsRoot, './report.html')
  }));
}
// 改用merge来合并配置
module.exports = merge(webpackConfig, {
  // ...configs
});
```

在`package.json`打包命令中增加参数：
```json
"scripts": {
  "build": "cross-env NODE_ENV=production node config/build.js --report"
},
```

运行`npm run build`，生成的dist目录中会有一个report.html文件，就是我们的分析报告。

# 支持less和css modules
现在我们使脚手架支持css，less和css modules：

先装工具：
```bash
$ npm i -D style-loader css-loader less less-loader
```

增加配置：
```javascript
// webpack.base.js
module: {
  rules: [
    {
      oneOf: [
        // ... configs
        {
          test: /\.(less|css)$/,
          use: [
            { loader: 'style-loader' },
            {
              loader: 'css-loader',
              options: {
                modules: false // 如果要启用css modules，改为true即可
              }
            },
            {
              loader: 'less-loader',
              options: { javascriptEnabled: true }
            }
          ]
        },
      ]
    }
  ]
}
```

# 提取css
我们发现打包好的文件中并没有css，但是css却可以正常工作，这是因为webpack并没有把样式从js中剥离出来。

为了方便管理静态资源，充分利用缓存，我们需要将css单独打包。

先安装工具：
```bash
$ npm i -D optimize-css-assets-webpack-plugin
```

增加打包配置：
```javascript
// webpack.prod.js
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');

// ...webpack configs
optimization: {
  minimizer: [
    new OptimizeCSSAssetsPlugin({
      cssProcessorOptions: true ? { map: { inline: false }} : {}
    })
  ]
}
```

运行打包命令，就能看到生成的css文件。

# 自动增加css前缀
使用[postcss](https://github.com/postcss/postcss)，可以自动为css增加浏览器前缀。

安装依赖：
```bash
$ npm i -D postcss-loader autoprefixer
```

增加webpack配置：
```javascript
// webpack.base.js，webpack.prod.js
{
  test: /\.(less|css)$/,
  use: [
    { loader: 'style-loader' },
    {
      loader: 'css-loader',
      options: {
        modules: false
      }
    },
    'postcss-loader', // 注意插入的位置，webpack.prod.js也要加这一项！！！
    {
      loader: 'less-loader',
      options: { javascriptEnabled: true }
    }
  ]
},
```

在根目录新建`postcss.config.js`：
```javascript
module.exports = {
  plugins: {
    autoprefixer: {}
  }
};
```

在`package.json`中增加配置：
```json
"browserslist": [
  "> 1%",
  "last 2 versions",
  "not ie <= 8",
  "iOS >= 8",
  "Firefox >= 20",
  "Android > 4.4"
]
```

# postcss-px-to-viewport示例
这里提供一个利用postcss做基于vh，vw布局的配置例子。

安装依赖：
```bash
$ npm i -D postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg
$ npm i -D cssnano cssnano-preset-advanced
```

修改`postcss.config.js`：
```javascript
module.exports = {
  plugins: {
    'postcss-aspect-ratio-mini': {}, // 处理元素容器的宽高比
    'postcss-write-svg': { //处理1px边框
      utf8: false
    },
    'postcss-px-to-viewport': {
      viewportWidth: 750,
      viewportHeight: 1334,
      unitPrecision: 3,
      viewportUnit: 'vw',
      selectorBlackList: ['.ignore', '.hairlines'],
      minPixelValue: 1,
      mediaQuery: false
    },
    cssnano: {
      'cssnano-preset-advanced': {
        zindex: false, // 这里一定要关掉，否则所有的z-index会被设为1
      }
    },
    autoprefixer: {}
  }
};
```

配置完成后，如果是基于750px宽度设计图，那么设计图上1px就直接在样式中写1px即可，打包时会自动转为vw单位。
