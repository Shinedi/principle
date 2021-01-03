#### 基本使用
1. computed和watch?
* computed有缓存,data不变则不会重新计算
* watch如何深度监听
```
info: {
    handler(oldVal, val){
        console.log(oldVal, val)
    },
    deep: true
}
```
* watch监听引用类型，拿不到oldval

2. class和style
* 使用动态属性
* 使用驼峰式写法
```
<p :class="{}">
<p :class="[]">
<p :class="styleData"> // styleData用驼峰写法
```

3. 条件渲染？
* v-if，v-show是将隐藏元素设置display：none
* 更新不频繁用v-if,频繁更换用v-show

4. 循环渲染
* v-for
* key的重要性
* v-for和v-if不能一起使用: v-for的优先级高于v-if

5. 事件
* 自定义参数`@click="increment(2, $event)"`
```
event.__proto__.constructor // 原生的event对象
event.target // 是在哪个对象上触发的事件
event.currentTarget // 事件是被挂载到当前元素
```

6. 事件修饰符
```
@click.stop
```

7. v-model
```
v-model.lazy="name"
v-model.trim="name"
```

8. 父子组件如何通讯
* props和$emit
* 组件间通讯-自定义事件
* 组件生命周期

9. 组件间通讯-自定义事件(兄弟组件(不是父子组件)如何通讯)
* a组件调用自定义事件`event.$emit('onAddTitle', this.title)` // event其实是vue的实例
* b组件绑定自定义事件`event.$on('onAddTitle', this.fn)`
* 销毁自定义事件: `event.$off('onAddTitle', this.fn)` // 及时销毁,否则可能造成内存泄露

10. 生命周期(单个组件)
* 挂载阶段: created(vue实例初始化完成，页面还没渲染) monted（页面渲染完成）
* 更新阶段：beforeUpdated,updated
* 销毁阶段: beforeDestroy(解除绑定、销毁子组件)

11. 生命周期（父子组件）
* parent.created -> child.created -> child.monted -> parent.monted
* parent.beforupdated -> child.beforupdated -> child.updated -> parent.updated 

12. 高级特性
* 自定义v-model
* $nextTick
* slot
* 动态、异步组件
* keep-alive
* mixin

13. 自定义v-model
```
<template>
    <input type="text" :value="text1" @input="$emit('change1', $event.target.value)">
    <!--
        1. 上面的input使用了:value而不是v-model
        2. 上面的change1和model.event要对应起来
        3. text1属性对应起来
    -->
</template>
<script>
export default {
    model: {
        prop: 'text1',
        event: 'change1'
    },
    prop: {
        text1: String,
        default(){
            return ''
        }
    }
}
</script>

父组件
{{name}}
<child v-model="name"></child>
```

14. $nextTick
* Vue是异步渲染
* data改变之后，DOM不会立刻渲染
* $nextTick会在DOM渲染之后被触发,以获取最新Dom节点

15. slot
* 基本使用
```
<template>
    <a :href="url">
        <slot>
            默认内容,即父组件没设置内容时，这里显示
        </slot>
    </a>
</template>
<script>
export default {
    props:['url'],
    data(){

    }
}
</script>

父组件
<slotCom :url='url'>
    hahaha
</slotCom>
```
作用域插槽(父组件向子组件的插槽中传递子组件的data值)
```
<template>
    <a :href="url">
        <slot :slotData="website">
            默认内容,即父组件没设置内容时，这里显示
        </slot>
    </a>
</template>
<script>
export default {
    props:['url'],
    data(){
        website: {
            url: '',
            title: 'ahaha',
            subTitle: '轻量级富文本'
        }
    }
}
</script>

父组件
<scopedSlotCom :url='url'>
    <template v-slot="slotProps">
        {{slotProps.slotData.title}}
    </template>
</scopedSlotCom>
```
<image src="./images/named_slot.png" />
16. 动态组件
* `:is="component-name"`用法
* 
```
<template>
    <component :is="nextTickName"/>
</template>
<script>
import NextTick from './NextTick'
export default{
    components: {
        NextTick
    },
    data(){
        return {
            nextTickName: 'NextTick'
        }
    }
}
</script>
```

17. 异步组件
* import()函数
* 按需加载,异步加载大组件
```
<template>
    <FormData v-if="showFormData"/>
    <button @click="showFormData=true">show form demo</button>
</template>
export default{
    components: {
       FormData: ()=> import('../BaseUse')
    }
}
```

18. keep-alive缓存组件
* 缓存组件
* 频繁切换，不需要重复渲染
* Vue常见性能优化
```
<keep-alive>
    <comA v-if="state=='A'">
    <comA v-el-if="state=='B'">
</keep-alive>
```

19. mixin(抽离公共逻辑)
* 多个组件有相同的逻辑，抽离出来
* mixin并不是完美的解决方案
* vue3 提出的compositon API旨在解决这些问题
缺点
* 变量来源不明确,不利于阅读
* 多mixin可能会造成命名冲突

20. vuex
* 基本概念： `state、getters、action、mutation`
* 用于vue组件： `dispatch、commit、mapState、mapGetters、mapActions、mapMutations`
* action中进行异步操作、mutation中进行整合

21. vue-router使用
* 路由模式(hash,h5History)
    hash模式（默认）,如：http://abc.com/#/user/100
    h5 histor模式：http://abc.com/user/100
    后者需要server端支持,因此无特殊要求可用前者
* 路由配置(动态路由、懒加载)
```
const router = new VueRouter({
    mode: 'history',
    routes: []
})
```
动态路由/懒加载

```
const User = {
    template: '<div>User{{$route.params.id}}</div>'
}
const router = new VueRouter({
    routes: [
        {path: '/user/:id', component: User}，
        {
            path: '/feedback',
            component: ()=>import('../../Base')
        }
    ]
})

```


