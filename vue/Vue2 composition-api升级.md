# Vue2 composition-api 升级

在 Vue2.x 版本的项目中，获得和 Vue3.0 中一样的开发体验。

`composition-api` 字面意思是组合 API，它是为了实现基于函数的逻辑复用机制而产生的。这也是 Vue3 亮点之一，那么我们如何才能够在 Vue2 项目中使用呢？



### 支持 composition-api

>  本插件要求使用 TypeScript **4.2** 或以上版本

安装`@vue/composition-api`依赖。

```shell
yarn add @vue/composition-api
```

在项目中添加以下配置

```js
import Vue from 'vue'
import VueCompositionApi from '@vue/composition-api'

Vue.use(VueCompositionApi)
```

更多内容：[vue/composition-api](https://github.com/vuejs/composition-api) 



### 支持`<script setup>`语法

`<script setup> `是在单文件组件 (SFC) 中使用组合式 API 的编译时语法糖，是 Vue3.2 新加入的语法。那么，我们也可以在 Vue2 项目中使用它。[文档](https://v3.cn.vuejs.org/guide/composition-api-setup.html)


需要安装`unplugin-vue2-script-setup`依赖。

```shell
yarn add unplugin-vue2-script-setup -D
```

这里使用的是`webpack`，所以需要在配置中加上

```js
// webpack.config.js
const ScriptSetup = require('unplugin-vue2-script-setup/webpack').default
module.exports = {
  /* ... */
  plugins: [
    ScriptSetup({ /* options */ }),
  ]
}
```

其他的一些打包工具使用方式，可以查看 [unplugin-vue2-script-setup](https://github.com/antfu/unplugin-vue2-script-setup)。



### 在 Vue2 项目中使用 Volar

以下是官方的解释：

> 我们建议将 VS Code 与 Volar 结合使用以获得最佳体验（如果您拥有 Vetur，您可能希望禁用它）。 使用 Volar 时，您需要安装 @vue/runtime-dom 作为 devDependencies 以使其在 Vue 2 上工作。

```shell
yarn add @vue/runtime-dom -D
```



### 支持 TypeScript 语法

#### tsconfig.json

`tsconfig.json`文件中指定了用来编译这个项目的根文件和编译选项。

这里需要注意 IDE 会提示缺少全局类型。那么需要添加以下配置

```
{
  "compilerOptions": {
    "types": [
      "unplugin-vue2-script-setup/types"
    ]
  }
}
```

Volar 优先支持 Vue 3。Vue 3 和 Vue 2 模板有些不同。项目中需要设置 ExperimentCompatMode 选项以支持 Vue 2 模板。

```
{
  "compilerOptions": {
    ...
  },
  "vueCompilerOptions": {
    "experimentalCompatMode": 2
  },
}
```

#### ESLint

如果项目中使用了 ESLint，可能会出现带有 <script setup> 的 `@typescript-eslint/no-unused-vars` 警告。

建议可以禁用它：在 `tsconfig.json` 中添加 `noUnusedLocals: true` 。Volar 会正确推断出真正不通过`ESlint` 校验的那一部分。



### unplugin-auto-import 自动按需引入

在 <script setup>需要手动 `import` 各种方法， [unplugin-auto-import](https://github.com/antfu/unplugin-auto-import) 会根据代码中使用到的内容，自动引入 `Vue` 各种方法

未使用插件前

```vue
<script setup>
import { computed, ref } from 'vue'
const count = ref(0)
const doubled = computed(() => count.value * 2)
</script>
```

使用之后

```vue
<script setup>
const count = ref(0)
const doubled = computed(() => count.value * 2)
</script>
```



#### 安装

```shell
npm i -D unplugin-auto-import
```



#### 使用

在 `webpack` 中使用，其他打包工具的配置可以在 `Github` 上找到。

```js
// webpack.config.js
module.exports = {
  /* ... */
  plugins: [
    require('unplugin-auto-import/webpack')({ /* options */ }),
  ],
}
```



#### ESlint配置

项目使用插件后，可能会出现 `no-undef` 的报错。那么需要在 .eslintrc.js 中添加以下配置

```js
// .eslintrc.js

module.exports = {
  /* ... */
  extends: [
    // ...
    './.eslintrc-auto-import.json',
  ],
}
```

注：`.eslintrc-auto-import.json` 是自动生成的。如果配置文件修改没有及时生效，请检查配置文件、重启 `ESlint` 服务或者编辑器。