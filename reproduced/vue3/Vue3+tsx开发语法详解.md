> 很多组件库都使用了 TSX 的方式开发，主要因为其灵活性比较高，TSX 和 SFC 开发的优缺点就不介绍了，这里主要说一下将 SFC 项目改造为 TSX 的过程。安装 JSX 库 pnpm install 

很多组件库都使用了`TSX`的方式开发，主要因为其灵活性比较高，`TSX`和`SFC`开发的优缺点就不介绍了，这里主要说一下将`SFC`项目改造为`TSX`的过程。

安装 JSX 库
--------

```
pnpm install @vitejs/plugin-vue-jsx -D
```

安装完之后在`vite.config.ts`进行插件使用，代码如下：

```
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import vueJsx from "@vitejs/plugin-vue-jsx";

export default defineConfig({
  plugins: [
    vue(),
    vueJsx() 
  ],
});

```

然后就可以愉快的使用`TSX`来开发`Vue`组件了，下面主要说一下`SFC`和`TSX`的部分区别。

基本语法对照 SFC
----------

### defineComponent 和 setup

SFC 方式结构固定：template、script、style

```vue
<template>
  <div>Hello World</div>
</template>

<script setup lang="ts">
</script>

<style scope>
</style>
```

TSX 方式就完全是一个 ts 文件的写法，没有模板 template 和样式 style

```vue
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    // Todo
    return () => <div>Hello World</div>
  }
})
```

setup 中函数的返回值有多种方式，可以直接返回 html：`<div>Hello World</div>`，这个适合结构简单的页面，如果返回比较多，可以使用如下方式：

```vue
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    // Todo
    return () => (
      <div>
        <div>Child1</div>
        <div>Child2</div>
        <div>Child3</div>
      </div>
    )
  }
})
```

如果是多节点，可以使用空符号包裹

```tsx
return () => (
  <>
    <div>Child1</div>
    <div>Child2</div>
    <div>Child3</div>
  </>
)
```

在以上的方式中我们把除了布局以外的逻辑都写在`//Todo`部分，但是有时候我们需要做一些按条件渲染的逻辑，那么也可以在`return`里加处理逻辑，例如：

```vue
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    // Todo
    return () => {
      if (something) {
        return (
          <div>
            <div>Child1</div>
            <div>Child2</div>
            <div>Child3</div>
          </div>
        )
      } else {
        return (
          <div>noChild</div>
        )
      }
    }
  }
})
```

这种方式类似`v-if`，但是和`v-if`还是有点区别，`v-if`可以作用在更小的范围，而这种方式只适合整个组件的条件渲染，这个可能不好理解，在下面`v-if`的使用中我们会看到区别。

### v-if

使用条件判断语句来实现 v-if 的功能，一般是三目运算符

```
<template>
  <div>
    <span v-if="condition ">A</span>
    <span v-else>B</span>
  </div>
</template>

return () => (
  <div>
    {
      condition ?  <span>A</span> : <span>B</span>
    }
  </div>
)
```

在这里你可以看到`v-if`的使用和我们上面的条件返回不一样，区别就是整体渲染没有大的变化，只是其中部分地方要按条件显示。

### v-bind

绑定变量，也就是简写的: 冒号，修改方式就是将冒号去掉，把双引号改为大括号

```
<template>
  <div :class="c_class" :style="c_style" :custom="custom">
</template>

// TSX
return () => (
  <div class={c_class} style={c_style} custom={custom}>
)
```

### v-for

采用 map 循环的方式，返回一个数组

```
<template>
  <div v-for="(index, item) in list" :key="item">{{item}}</div>
</template>


return () => (
  <>
    {
      list.map((item, index) => <div>{item}</div>
    }
  </>
)

```

### 自定义指令

自定义指令和普通指令`v-model`一样

```
<template>
  <div v-custom="command">自定义指令</div>
</template>


return () => (
  <div v-custom={command}>自定义指令</div>
)
```

### 插槽

插槽有两种实现方式，一种是用`v-slots`绑定对象，一种是直接在元素中使用对象。

```
<template>
  <div>
    <slot>默认插槽: default</slot>
    <br />
    <slot >具名插槽：header</slot>
    <br />
    <slot >作用域插槽：main</slot>
  </div>
</template>

// SFC parent
<template>
  <div>
    <child>
      我就是默认的
      <template #main="row"> 我是主要的{{ row.var1 }} </template>
      <template #header>我是头</template>
    </child>
  </div>
</template>


import { defineComponent } from 'vue';

export default defineComponent({
  setup(props, { slots }) {
    return () => (
      <div>
        默认插槽: {slots.default && slots.default()}
        <br />
        具名插槽: {slots.header && slots.header()}
        <br />
        作用域插槽:{slots.main && slots.main({ name: '我是作用域插槽的传值' })}
      </div>
    );
  }
});

// TSX parent 第一种方式
return () => (
  <Child v-slots={{
      default: () => '默认的内容是',
      header: () => '我是有名称的',
      main: (props: Record<'name', string>) => '我才是主要的' + props.name
    }}>
  </Child>
)


return () => (
  <Child v-slots={{
      default: () => '默认的内容是',
      header: () => '我是有名称的',
      main: (props: Record<'name', string>) => '我才是主要的' + props.name
    }}>
  </Child>
)
```

### props

父组件向子组件传值

```
defineProps<{
  name: string,
  childs: string[]
}>()


export default defineComponent({
  props: {
    name: String,
    childs: {
      type: Array as PropsType<string[]>,
      default: []
  },
  setup(props, { slots }) {
    return () => <div>{props.name}</div>
  }
})
```

需要注意的是，prop 传递过来的值如果没有默认值，需要判断是否为空，可以使用计算属性或者条件渲染处理。

### emit

子组件向父组件传值

```
const emits = defineEmits<{
  (e: 'changeName', name: string): void;
}>();

emits('changeName', '张三')


export default defineComponent({
  emits: ['changeName'],
  setup(props, {emit}) {
    emit('changeName', '张三')
    return () => <div></div>
  }
})
```

### 事件监听

事件监听就是`v-on`或者`@`，在`TSX`中事件以`on`开头，即使我们的自定义事件没有`on`，也要在监听的时候加上，一般都采用的是小驼峰的方式。

```
// SFC
<template>
  <div @click="handleClick">无参数</div>
  <div @click="(event) => handleClick1(event)">鼠标事件参数</div>
  <div @click="handleClick2('abc')">自定义参数</div>
</template>

// TSX
return () => (
  <>
    <div onClick={handleClick}>无参数</div>
    <div onClick={(event) => handleClick1(event)}>鼠标事件参数</div>
    <div onClick={() => handleClick2('abc')}>自定义参数</div>
  </>
);

// 函数定义相同
const handleClick = () => {
  console.log('click');
};

const handleClick1 = (e: MouseEvent) => {
  console.log(e.offsetX);
};

const handleClick2 = (name: string) => {
  console.log(name);
};
```

自定义事件只需要在事件名前面加上`on`即可，参数传递与上面一致

```
<div @custom="handleCustom()"></div>

// TSX
<div onCustom={handleCustom}></div>
```

在`TSX`中处理事件不能使用事件修饰符，因此需要在事件函数中自行处理，例如冒泡、阻止默认行为等。

### 属性 / 事件继承

对于这个我也不知道怎么描述，当我们给一个组件传递属性和事件时，一般子组件在`props`中接收属性值，`emits`中接收事件，但是我们也可以传一些额外的属性和事件，即不在`props`和`emits`中的属性和事件，虽然这是不推荐的做法，但是有时候当我们封装第三方库的时候，这种用法就非常的方便。具体看如下代码：

```
<Child  @click="handleClick" />


<div v-bind="$attrs">继承属性/事件</div>

// TSX child
<div {...attrs}>继承事件</div>
```

`SFC`中，在`template`中我们可以通过`$attrs`获取到额外的属性和方法，`script`中可以通过`getCurrentInstance`方法获取组件对象，然后通过`.attrs`拿到属性和方法。

`TSX`中，直接通过`attrs`获取属性和方法，通过`{...attrs}`把属性和方法传递给子元素。

### 其他命令

`v-show`和`v-model`与`SFC`中使用一样，这里不做示例

### 组件引用

通过`ref`获取组件`dom`信息

```
<ChildComponent ref="com" />

const com = ref(null)


const com = ref(null)

return () => <ChildComponent ref={com} />

```

### 对外暴露属性和方法

在父组件中直接调用子组件的属性和方法

```
defineExpose<{
  name: Ref<string>;
  handleClick: () => void;
}>({
  name,
  handleClick
});

setup(props, { expose }) {
  expose({name, handleClick})
}

```

### 样式修改

样式的改造一度是我切换`TSX`的最大痛点，因为在`SFC`中最麻烦的是修改第三方库的样式，一般要用到`:deep`，而且有时候还不一定成功，非常麻烦，改为`TSX`后我一直不知道怎么解决这种问题，后来搞定以后再回过头来看，发现是`vue`写久了养成了固定思维。我们在`vue`文件中写的样式都包含在`scoped`下面，如果不加`scoped`就可能会造成全局样式污染。那为什么会造成全局样式污染？又为什么加了`scoped`就不会呢？实际上我们只要知道`CSS`基础，明白`CSS`中的样式优先级即可。`vue`生成的项目最终还是会回归到`html`、`css`、`js`来，因此我们从这里来理解就方便多了。

*   为什么会造成全局样式污染？
    

这个不是`vue`的专利，而是`css`本身的优先级问题，就是如果我们定义了相同的`css`类，并以相同的方式来使用它，那么根据先后加载顺序，就会导致后加载的覆盖掉先加载的样式，造成先加载的样式无效，这就是所谓的样式污染。

*   为什么加了`scoped`就不会造成样式污染呢？
    

我们看一个简单的例子：

```
<div class="item">样式示例</div>

<style lang="less" scoped>
.item {
  background-color: pink;
}
</style>

```

`vue`组件在渲染的时候，会给元素增加一个属性`data-v-xxxx`，然后在生成样式的时候也会在样式上加上`[data-v-xxxxx]`，这是`css`属性选择器的用法，这样根据`css`选择器的优先级，这个属性就具有唯一性。

但是在`TSX`中没有了`scoped`怎么办？很简单，回归原始的`css`即可。在原始`css`中需要我们自己来保证`css`选择的唯一性，具体做法就是给组件内使用的 css 类都加上唯一前缀，例如组件名称为`Child`，那么所有的`css`类都加上`child-xxx`，因为我们肯定要保证组件名称的唯一性，所以这样下来对应的样式也就是唯一的。这就要求我们给所有需要修改样式的元素都加上类或者自定义属性，以便于我们可以通过唯一的`css`选择器选中它。

示例如下：

创建一个`css`文件：`child.css`

```
.child-item {
  background-color: pink;
}
```

在`tsx`文件中引用

```
// TSX child.tsx
import './child.css'

return () => <div class="child-item"></div>
```

除了上面这种保证样式名称唯一的方式以外，`vue`其实一直为我们提供了另外一种方式 -`css module`，具体来讲就是把`css`作为模块引入到`js`中，然后会生成一个唯一的名称，在以前用`webpack`的时候还需要装额外的包，现在`vite`已经帮我们集成了，只需要在`vite.config.ts`中加一下配置即可。

```
css: {
  modules: {
    localsConvention: 'camelCase'
  }
}
```

这里规定`css`类名的命名规则为小驼峰，即`child-item`类在`js`中会变成`childItem`变量。但是要实现`css module`的功能，对`css`文件命名由要求，必须在后缀名前面是`module`，例如`xxx.module.css`、`xxx.module.less`、`xxx.module.scss`。

示例如下：

创建一个`css`文件：`child.module.css`

```
.child-item {
  background-color: pink;
}
```

在`tsx`文件中引用

```
import styles from './child.module.css'

return () => <div class={styles.childItem}></div>
```

元素上绑定的`css`和全局的`css`都出现了变化，这种方式我们就不需要去关注编写的`css`是否是唯一的，`vite`会帮我们自行处理，只是在使用的时候有一些区别。

除了常规的`css`使用，我们还有动态`class`的使用。

```
<div class="box" :class="{active: count === 1}"></div>

// TSX
<div class={['box', count===1 ? 'active' : '']}></div>
```

我们把需要的`class`处理成一个数组给它即可。

除了动态`class`还有动态`style`的使用。

```
// SFC
<div :style="{ height: height + 'px' }"></div>

// TSX
<div style={{ height: height.value + 'px' }}></div>
```

总结
--

目前对于大部分场景的使用都写到了，如果后期有其他的语法糖在进行补充，示例项目：vue3-tsx-todo，请在 gitee 上进行搜索

