# 用 Webpack 优化第三方库

> 今天又是快乐的一天呢~

项目中经常会使用到第三方的库，通过本文的一些小技巧可以将你的项目在 `webapck` 打包之后变得更小。

## babel-polyfill



## lodash

`lodash` 提供了许多的工具函数供开发者使用。



### 使用 `babel-plugin-lodash`

`babel-plugin-lodash` 将 `lodash` 的全部导入替换为特定 `lodash` 函数的导入：

```js
// 不推荐
import _ from 'lodash';
_.map([1, 2, 3], i => i + 1);

// 推荐
import map from 'lodash/map';
map([1, 2, 3], i => i + 1);
```

注意：**这种引入方式不能链式调用，待优化**

```js
// 暂不支持
import map from 'lodash/map';
map([1, 2, 3], i => i + 1).value();
```



### 给 `lodash `  起个别名 `lodash-es`

项目中的某些依赖项可能使用 `lodash-es` 包而不是 `lodash`。如果是这种情况，`lodash` 将被绑定两次。

可以给 `lodash `  起个别名叫 `lodash-es`

```js
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      'lodash-es': 'lodash',
    },
  },
};
```



### 使用 `lodash-webpack-plugin`

`lodash-webpack-plugin` 去掉了你可能不需要的部分 `lodash` 功能。例如，如果项目使用 `_.get()` 但不需要深度路径支持，则此插件会将其删除。将它添加到你的 `webpack` 配置中可以使包更小。

**谨慎使用本插件**，它默认设置剔除了许多功能，而你的项目中可能正好使用到了某一些功能，所以建议在做好分析的情况下进行使用，新项目除外。



## lodash-es

`lodash-es` 是带有 ES 导入和导出的 `lodash`。