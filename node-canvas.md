canvas 预编译了一些常见系统的二进制码，当没有找到匹配的预编译码时，会自动fallback执行编译（npm install --build-from-source），编译时各种报错，分析后认为是缺少依赖所致，但是从何而知缺少了哪些依赖呢，第一反应是在项目的 issue 搜索报错相关内容，查看后发现并没有特别有价值的解法，随后看到一条 issue，想让作者提供一个 dockerfile 以供容器中使用，于是想到作者可能编写了 ci 相关内容。

查看项目根目录，果然发现有一个 ci 文件 [.travis.yml](https://github.com/Automattic/node-canvas/blob/master/.travis.yml) 
里面列出了用到的依赖为 libcairo2-dev libjpeg8-dev libpango1.0-dev libgif-dev librsvg2-dev g++-4.9，通过查询资料，得知 alpine 的 apk 包管理工具中对应的依赖名分别为：
- g++
- cairo-dev
- jpeg-dev
- pango-dev
- giflib-dev

如果是 ubuntu 等操作系统，可以用 apt-get，yum 等工具安装。

由于需要编译，增加 build-base 库，最后得到dockerfile命令如下：
```dockerfile
RUN npm install -g node-gyp \
  && echo "http://mirrors.aliyun.com/alpine/v3.7/main" > /etc/apk/repositories \
  && echo "http://mirrors.aliyun.com/alpine/v3.7/community" >> /etc/apk/repositories \
  && apk update \
  && apk add --no-cache build-base g++ cairo-dev jpeg-dev pango-dev giflib-dev \
  && npm install canvas
```

> ⚠️ 注意：
> 如果依赖已经装全任无法成功安装，可以尝试调整 nodejs 或 canvas 版本。

安装完毕之后，运行项目测试绘图，报 `symbol not found` 错误，搜索 issue 和错误提示后，猜测是和项目中已有的 shape.js 冲突导致，移除全部 shape 相关代码，测试绘图，成功运行。

> ❤️ 总结：
> 1. 读取预编译文件时会执行编译
> 2. 编译需要用到 g++ ，make，node-gyp
> 3. 需要完整安装各种依赖
> 4. 与 shapre.js 不能共存 (canvas@2.6.0)
> 5. python 版本必须是 2.7

# Windows 10 安装
如果读取不到预编译文件，在 win10 下依然需要编译，通过搜索可以得到各种解法，以下提供一种较简单的编译安装方法。

## 安装 node-gyp
这是必备组件，需要全局安装：
```shell
$ npm i -g node-gyp
```
## 安装 chocolatey 和依赖
[chocolatey](https://chocolatey.org/) 是 windows 下的包管理工具，可以用它下载管理各种软件。

访问官网查询安装方式，提供了 cmd 和 powershell 两种安装方式。

安装好后，继续安装依赖：
```shell
$ choco install -y python2 gtk-runtime microsoft-build-tools libjpeg-turbo
```

## 安装 GTK 工具
canvas 依赖 cairo ，GTK 中包含了 cairo，因此我们需要安装 GTK2，去 GTK 官网下载，然后解压缩到 `C:\GTK`。如果不想解压到 c 盘，那么需要修改路径：
```shell
$node-gyp rebuild --GTK_Root=C:\\somewhere\\GTK
```

> ⚠️ 注意：
> GTK3 中没有 libpng，因此必须选择 GTK2。

接下来运行 `npm i canvas`，应该可以顺利安装。

> ❤️ 总结：
> 1. 需要 node-gyp 来编译
> 2. 用 choco 安装依赖比较方便，py 还是 2.7
> 3. GTK2，而不是3
> 4. 如果不用 choco 安装依赖，可以装个 visual studio，或者用 npm 全局安装 windows-build-tools
