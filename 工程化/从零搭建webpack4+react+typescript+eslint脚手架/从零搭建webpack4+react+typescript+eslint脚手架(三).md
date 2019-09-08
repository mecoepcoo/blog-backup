# 处理静态资源
js的打包基本处理完了，还有图片、音频等静态资源需要处理。

依然先装依赖：
```bash
$ npm i -D url-loader file-loader
$ npm i -D @svgr/webpack # 顺带支持一下导入svg图片
```

增加webpack配置：
```javascript
// webpack.base.js
{
  test: /\.svg$/,
  use: ['@svgr/webpack']
},
{
  test: /\.(jpg|jpeg|bmp|png|webp|gif)$/,
  loader: 'url-loader',
  options: {
    limit: 8 * 1024, // 小于这个大小的图片，会自动base64编码后插入到代码中
    name: 'img/[name].[hash:8].[ext]',
    outputPath: config.assetsDirectory,
    publicPath: config.assetsRoot
  }
},
// 下面这个配置必须放在最后
{
  exclude: [/\.(js|mjs|ts|tsx|less|css|jsx)$/, /\.html$/, /\.json$/],
  loader: 'file-loader',
  options: {
    name: 'media/[path][name].[hash:8].[ext]',
    outputPath: config.assetsDirectory,
    publicPath: config.assetsRoot
  }
}
```

> tips: 生产环境需要合理使用缓存，需要拷贝一份同样的配置在`webpack.prod.js`中，并将name中的`hash`改为`contenthash`

接下来我们要把`public`目录里除了`index.html`以外的文件都拷贝一份到打包目录中：

安装依赖：
```bash
$ npm i -D copy-webpack-plugin
```

增加配置：
```javascript
// webpack.base.js
const CopyWebpackPlugin = require('copy-webpack-plugin');

plugins: [
  // ...other plugins
  new CopyWebpackPlugin([
    {
      from: 'public',
      ignore: ['index.html']
    }
  ])
]
```

# 提取公共模块，拆分代码
有些模块是公共的，如果不把他拆分出来，那么他会在每一个被引入的模块中出现，我们需要优化与此相关的配置。

```javascript
// webpack.prod.js
entry: {
  app: './src/index.tsx',
  vendor: ['react', 'react-dom'] // 不变的代码分包
},
optimization: {
  splitChunks: {
    chunks: 'all',
    minChunks: 2,
    maxInitialRequests: 5,
    cacheGroups: {
      // 提取公共模块
      commons: {
        chunks: 'all',
        test: /[\\/]node_modules[\\/]/,
        minChunks: 2,
        maxInitialRequests: 5,
        minSize: 0,
        name: 'common'
      }
    }
  }
}
```

# 压缩代码（js和css），gzip
通过使用打包分析工具，我们会发现打出来的包都很大，远不能满足生产环境的体积要求，因此还需要对代码进行压缩。

安装依赖：
```bash
$ npm i -D uglifyjs-webpack-plugin mini-css-extract-plugin compression-webpack-plugin
```

增加和修改配置：
```javascript
// webpack.prod.js
const UglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CompressionWebpackPlugin = require('compression-webpack-plugin');

{
  test: /\.(less|css)$/,
  use: [
    MiniCssExtractPlugin.loader, // 注意书写的顺序
    {
      loader: 'css-loader',
    },
    'postcss-loader',
    {
      loader: 'less-loader',
      options: {
        javascriptEnabled: true,
      }
    }
  ]
},
// ...configs
plugins: [
  new HtmlWebpackPlugin({
    template: config.indexPath,
    minify: {
      removeComments: true,
      collapseWhitespace: true,
      removeRedundantAttributes: true,
      useShortDoctype: true,
      removeOptionalTags: false,
      removeEmptyAttributes: true,
      removeStyleLinkTypeAttributes: true,
      removeScriptTypeAttributes: true,
      removeStyleLinkTypeAttributes: true,
      removeAttributeQuotes: true,
      removeCommentsFromCDATA: true,
      keepClosingSlash: true,
      minifyJS: true,
      minifyCSS: true,
      minifyURLs: true,
    }
  }),
  new MiniCssExtractPlugin({
    filename: 'css/[name].[contenthash:8].css'
    // chunkFilename: '[name].[contenthash:8].chunk.css'
  }),
  // gzip压缩
  new CompressionWebpackPlugin({
    filename: '[path].gz[query]',
    algorithm: 'gzip',
    test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'),
    threshold: 10240, // 大于这个大小的文件才会被压缩
    minRatio: 0.8
  }),
],
optimization: {
  minimizer: [
    new UglifyjsWebpackPlugin({
      sourceMap: config.productionJsSourceMap
    })
  ]
}
```

运行打包命令，查看打包好的文件，可以看到代码都被压缩好了。
