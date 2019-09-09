这一篇主要介绍代码规范相关的内容。

# eslint
我们通常使用lint工具来检查代码不规范的地方，以下是将 eslint、typescript 和 webpack 结合使用的例子。

首先安装依赖：
```bash
$ npm i -D eslint babel-eslint eslint-loader eslint-plugin-jsx-control-statements
$ npm i -D eslint-plugin-react @typescript-eslint/parser @typescript-eslint/eslint-plugin 
```

然后在根目录新建eslint配置文件`.eslintrc.js`：
```javascript
module.exports = {
  "root": true,
  "env": {
    "browser": true,
    "node": true,
    "es6": true,
    // "jquery": true
    "jest": true,
    "jsx-control-statements/jsx-control-statements": true // 能够在jsx中使用if，需要配合另外的babel插件使用
  },
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "sourceType": 'module',
    "ecmaFeatures": {
      "jsx": true,
      "experimentalObjectRestSpread": true
    }
  },
  "globals": {
    // "wx": "readonly",
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:jsx-control-statements/recommended", // 需要另外配合babel插件使用
  ],
  "settings": {
    "react": {
      "version": "detect" // 自动读取已安装的react版本
    }
  },
  "plugins": ["@typescript-eslint", "react", "jsx-control-statements"],
  "rules": {
    "no-extra-semi": 0, // 禁止不必要的分号
    "quotes": ['error', 'single'], // 强制使用单引号
    "no-unused-vars": 0 // 不允许未定义的变量
    // ...你自己的配置
  }
};
```

我们可能希望检查或不检查某些特定的文件，可以在根目录新建`.eslintignore`，以下配置不检查src目录以外的js文件：
```
**/*.js
!src/**/*.js
```

还需要配置webpack，才能在开发时启用eslint：
```javascript
// webpack.base.js
module: {
  rules: [
    // 把这个配置放在所有loader之前
    {
      enforce: 'pre',
      test: /\.tsx?$/,
      exclude: /node_modules/,
      include: [APP_PATH],
      loader: 'eslint-loader',
      options: {
        emitWarning: true, // 这个配置需要打开，才能在控制台输出warning信息
        emitError: true, // 这个配置需要打开，才能在控制台输出error信息
        fix: true // 是否自动修复，如果是，每次保存时会自动修复可以修复的部分
      }
    }
  ]
}
```

# prettier
除了约束开发时的编码规范外，我们一般还希望在提交代码时自动格式化代码，但我们只希望处理当前提交的代码，而不是整个代码库，否则会把提交记录搞得乱七八糟，[prettier](https://github.com/prettier/prettier)和[lint-staged](https://github.com/okonet/lint-staged)可以完成这项任务。

先安装工具：
```bash
$ npm i -D prettier eslint-plugin-prettier eslint-config-prettier
$ npm i -D lint-staged
```

在根目录增加prettier配置`.prettierrc.js`，同样的也可以增加忽略配置`.prettierignore`（建议配置为与lint忽略规则一致）：
```javascript
// 这个配置需要与eslint一致，否则在启用 eslint auto fix 的情况下会造成冲突
module.exports = {
  "printWidth": 120, //一行的字符数，如果超过会进行换行，默认为80
  "tabWidth": 2,
  "useTabs": false, // 注意：makefile文件必须使用tab，视具体情况忽略
  "singleQuote": true,
  "semi": true,
  "trailingComma": "none", //是否使用尾逗号，有三个可选值"<none|es5|all>"
  "bracketSpacing": true, //对象大括号直接是否有空格，默认为true，效果：{ foo: bar }
};
```

修改eslint配置`.eslintrc.js`：
```javascript
module.exports = {
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:jsx-control-statements/recommended", // 需要另外配合babel插件使用
    "prettier" // 注意顺序
  ],
  "plugins": ["@typescript-eslint", "react", "jsx-control-statements", "prettier"], // 注意顺序
  "rules": {
    "prettier/prettier": 2, // 这样prettier的提示能够以错误的形式在控制台输出
  }
};
```

然后我们要配置`lint-staged`，在提交代码时自动格式化代码。

修改`package.json`：
```json
"husky": {
  "hooks": {
    "pre-commit": "lint-staged"
  }
},
"lint-staged": {
  "src/**/*.{jsx,js,tsx,ts}": [
    "prettier --write",
    "eslint --fix",
    "git add"
  ]
}
```

# 用editorconfig统一编辑器规范
有些编辑器能够根据配置提示会自动格式化代码，我们可以为各种编辑器提供一个统一的配置。

在根目录新建`.editorconfig`即可，注意不要与已有的lint规则冲突：
```
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```

# 使用jest
使用jest可以帮助我们测试代码，在项目中使用jest的实现方式有很多种，文本不具体展开讨论，只提供一些必备的工具和配置。

必备工具：
```bash
$ npm i -D jest babel-jest ts-jest @types/jest
```

参考配置`jest.config.js`，测试文件均放在`__test__`目录中：
```javascript
module.exports = {
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
  testRegex: '(/__tests__/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$',
  moduleFileExtensions: ['ts', 'tsx', 'js', 'jsx', 'json', 'node'],
};
```
