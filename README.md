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
    "postcss-preset-env": {},
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
          { loader: "css-loader" },
          { loader: "postcss-loader" },
        ],
      },
      {
        test: /\.less$/i,
        use: [
          { loader: MiniCssExtractPlugin.loader },
          { loader: "css-loader" },
          { loader: "postcss-loader" },
          { loader: "less-loader" },
        ],
      },
    ],
  },
};
```
