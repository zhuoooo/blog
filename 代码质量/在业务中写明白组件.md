# 业务中写明白组件

> 今天又是快乐的一天呢~ 

![组件化图](./img/components.png)

## 写在前面

这个内容，相关联的文章、书籍有好多。每个开发者会通过编码、总结、再提升，是一个良性螺旋上升的过程；最近在小组内部进行的 `code review` 以及翻看自己所负责项目的组件代码时，有点想法和大家分享一下。本篇文章的内容方法都是个人和开发者讨论过，觉得相对合理的解决方法，如果你有更好的思路欢迎分享。

原本想着写一个《xxx组件化》、《xxx模块化》分享内容的，但在查阅资料的过程中，发现内容越来越虚无缥缈，拉都拉不回来，感叹水平不够，还是先分享一些平时项目开发遇到的问题吧。hhh



## 日常故事

>  “咦，这个组件的实现好厉害...多看两眼”
>
> “诶，这个组件写得比我还烂...看到 commit 记录人是自己（不愧是我~）”

你属于前者开发还是后者开发，还是处于薛定谔的状态：只有到写代码的时候才会从一种情况坍塌到另一种情况？



## 遇过问题

所有的代码片段都是删除了其他逻辑的，只是提供一个思考的案例，但都是真实案例。

对于代码实现，坚持没有“银弹”的原则，即使再好的代码片段放在不适宜的场景，它依旧是不合格的代码。而且一个功能的实现远不止一种方式，我们要选的是**相对合理**、**符合项目规范**的实现。



### 功能：附件列表 - 名称显示可点击下载组件

代码片段：

```vue
<template>
    <span
        v-for="(item, index) of data"
        :key="index"
        :class="item.canDownload ? '' : 'disable'"
        @click="downloadFile(item)"
    >
        {{ item[displayField] }}
    </span>
</template>

<script lang="ts" setup>
interface CompProps {
    data: Record<string, any>[];
    displayField: string;
    valueField: string;
}
const props = withDefaults(defineProps<CompProps>(), {
    displayField: 'name', // 问题1
    valueField: 'id'
});

const downloadFile = (file: Record<string, any>) => {
    // 判断该附件是否可以下载
    if (file.canDownload) {
        // 问题2、3
        // 文件下载工具函数
    }
};

// 给外部调用的下载全部接口
const downloadAllFile = () => {
    props.data.forEach(downloadFile);
};

defineExpose({
    downloadAllFile
});
</script>
```

存在的问题：

1. 冗余的配置 `displayField`、`valueField` 的存在

   这种拓展乍一看好像也没有什么不合理的地方，这样子以后还能支持改字段名显示的功能，也挺好的。但如果 `data` 数组中的数据有的要用 `name`、 有的用 `label` 进行展示呢？

   那么你一定想到了先在外边格式化一下 `data` ，既然都要格式化了，那为什么不干脆就**限制它的 `data` 数据格式为`{name:'',id: ''}` 还能少写点代码。**

2. 由于不确定展示的字段名，导致了 `data` 的声明不清晰，连组件内部都不知道它有什么样的字段。

3. 在一个功能函数 `downloadFile` 中，去做过滤操作，导致函数功能不单一

   函数功能不单一？这挺单一的了啊，就是进行下载文件的操作，它也没有干其他的事啊。

   但其实它 `if` 语句会造成无法下载的这个行为，所以建议**把 `if` 过滤性的判断放到外边去做**。这个可以是使得单测编写变得简单，如果要覆盖到 `if` 中的代码，那么就需要添加多一个案例。
   
   这里有一个小细节，就是 `template` 中也使用到了 `downloadFile` 这个函数，难道要在 `template `中去写 `if` 吗？这里有个小技巧，因为不能下载的附件是有 `disable` 的类的，那么通过 `pointer-events: none;` 就能阻断 `click` 事件。
   
   

改后代码：

```vue
<template>
    <span
        v-for="(item, index) of data"
        :key="index"
        :class="item.canDownload ? '' : 'disable'"
        @click="downloadFile(item)"
    >
        {{ item.name }}
    </span>
</template>
<style>
.disable {
    pointer-events: none;
}
</style>
<script lang="ts" setup>
interface Attachment {
    name: string;
    id: string;
    canDownload: boolean;
}
interface CompProps {
    data: Attachment[];
}
const props = defineProps<CompProps>();

const downloadFile = (file: Attachment) => {
    // 文件下载工具函数
};

// 给外部调用的下载全部接口
const downloadAllFile = () => {
    props.data.filter(item => item.canDownload).forEach(downloadFile);
};

defineExpose({
    downloadAllFile
});
</script>
```



### 场景：步骤条容器展示组件

代码片段：

```vue
<template>
    <ElSteps>
        <template v-for="(item, idx) of list">
            <!-- 问题一 hide 放到外边处理，好一点 -->
            <!-- 问题二 基础组件的配置项可以通过v-bind -->
            <ElStep
                v-if="!item.hide"
                :tilte="item.title"
                :icon="item.icon"
                :status="item.status"
                :key="idx"
            >
                <template #description>
                    <template v-if="item.empty">
                        <!-- 问题三 这里直接用固定的组件了，对后续扩展不友好 -->
                        <ElEmpty></ElEmpty>
                    </template>
                    <template v-else>{{ item.description }}</template>
                </template>
            </ElStep>
        </template>
    </ElSteps>
</template>

<script lang="ts" setup>
import { ElSteps, ElStep, ElEmpty } from 'element-plus';

type StepItem = {
    title?: string;
    icon?: string;
    status?: string;
    description?: string;
    empty?: boolean;
    hide?: boolean;
};

interface BusinessSteps {
    list: StepItem[];
}

const props = defineProps<BusinessSteps>();
</script>
```

存在的问题：

1. hide 放到外边处理，好一点。这一点其实放在数据处理就行了，在`list`中直接把要隐藏的过滤就好了
2. 第二个问题，在平时中也经常遇到，基于已有的组件进行封装的时候，会把基础的`props`给原封不动的给撸一遍。这里建议直接使用`v-bind` 代替，不用每次都给写一遍，还好扩展
3. 第三个问题，这里直接把组件写固定了，后续如果要支持别的组件很容易就演变成 `v-if-else`，所以建议这里使用动态组件`component` 支持外部传组件进来。



改后代码：

```vue
<template>
    <ElSteps>
        <ElStep v-for="(item, idx) of list" v-bind="item" :key="idx">
            <template #description>
                <template v-if="typeof item.desc === 'string'">
                    {{ item.desc }}
                </template>
                <template v-else>
                    <component :is="item.desc" />
                </template>
            </template>
        </ElStep>
    </ElSteps>
</template>

<script lang="ts" setup>
import { ElSteps, ElStep } from 'element-plus';
import { VueElement } from 'vue';

type StepItem = {
    title?: string;
    icon?: string;
    status?: string;
    desc?: string | VueElement;
};

interface BusinessSteps {
    list: StepItem[];
}

const props = defineProps<BusinessSteps>();
</script>
```



今天的分享就到这里了，如果觉得内容有对你起到一些帮助，可以点个免费的赞吗？谢谢~
