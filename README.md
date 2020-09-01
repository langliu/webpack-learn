# Webpack 配置说明

## 提取 CSS 成单独文件

将 CSS 提取到单独的文件中，为每个包含 CSS 的 JS 文件创建一个 CSS 文件，并且支持 CSS 和 SourceMaps 的按需加载。

```shell
npm install --save-dev mini-css-extract-plugin
```

webpack.config.js

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const devMode = process.env.NODE_ENV !== "production";

module.exports = {
  plugins: [new MiniCssExtractPlugin(
     filename: devMode ? "css/[name].css" : "css/[name].[hash].css",
     chunkFilename: devMode ? "css/[id].css" : "css/[id].[hash].css",
  )],
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
};
```

## CSS 兼容性配置

```shell
npm install --save-dev postcss-loader postcss-preset-env cssnano
```

postcss.config.js

```js
module.exports = {
  plugins: {
    'postcss-preset-env': {},
    cssnano: {},
  },
};
```

.browserslistrc

```text
# Browsers that we support

defaults
>.2% or last 2 versions
not IE 11
not IE_Mob 11
maintained node versions
```

webpack.config.js

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          { loader: MiniCssExtractPlugin.loader },
          { loader: 'css-loader' },
          { loader: 'postcss-loader' },
        ],
      },
      {
        test: /\.less$/i,
        use: [
          { loader: MiniCssExtractPlugin.loader },
          { loader: 'css-loader' },
          { loader: 'postcss-loader' },
          { loader: 'less-loader' },
        ],
      },
    ],
  },
};
```

## CSS 代码压缩

CSS 代码压缩有两种方式：

1. 通过 postcss 插件 `cssnano` 进行压缩
2. 通过 webpack 插件 `optimize-css-assets-webpack-plugin` 进行压缩

postcss.config.js

```js
module.exports = {
  plugins: {
    cssnano: {},
  },
};
```

webpack.config.js

```js
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');

module.exports = {
  plugins: [new OptimizeCssAssetsWebpackPlugin()],
};
```

## 启用 ESLint 检查 JavaScript 语法

```shell
npm install --save-dev eslint eslint-loader eslint-plugin-import eslint-config-airbnb-base
```

这里我们采用的是爱彼迎的规则 <https://github.com/airbnb/javascript>

.eslintrc

```json
{
  "extends": "airbnb-base"
}
```

webpack.config.js

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/i,
        exclude: /node_modules/,
        use: [
          {
            loader: 'eslint-loader',
            options: {
              cache: true,
            },
          },
        ],
      },
    ],
  },
};
```

## JavaScript 兼容性处理

在项目中我们可能经常会用到 ES2015 及之后的语法去编写代码，可是在老旧的浏览器（例如 IE 浏览器）中并不支持这些语法，如果我们需要兼容这些浏览器，那我们就需要将代码转换成这些老旧浏览器能够识别的语法。

Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。

```shell
npm install -D babel-loader @babel/core @babel/preset-env
```

.babelrc

```json
{
  "presets": ["@babel/preset-env"]
}
```

webpack.config.js

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.m?js$/i,
        exclude: /node_modules/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true,
            },
          },
          {
            loader: 'eslint-loader',
            options: {
              cache: true,
            },
          },
        ],
      },
    ],
  },
};
```

> 注意：如果项目中配置了 `.browserlistrc` ， babel 会根据 `.browserlistrc` 中的配置来进行转换，也就是说如果你在 `.browserlistrc` 中配置了不支持 IE 浏览器，则代码中的 `const` 之类的也语法不会转为 `var` 语法。

使用上面的配置可以转换 JavaScript 的基本语法，但是像 `Promise` 之类的语法还是不能够进行转换，这里我们需要在使用到一个库 `@babel/polyfill`。

可以将 `@babel/polyfill` 直接引入文件，在入口文件处 `import '@babel/polyfill'` ，但是这样直接引入会导致文件打包体积过大，更好的方法是按需引入。

.babelrc

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": 3,
        "targets": {
          "chrome": 60,
          "firefox": 60,
          "ie": 10
        }
      }
    ]
  ]
}
```

上面的配置告诉 babel 转换 JavaScript 代码时需要兼容 Chrome 浏览器的 60 版本、Firefox 浏览器的 60 版本以及 IE 浏览器的 10 版本。
