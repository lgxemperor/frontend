#### slot

``` javascript
<!-- TidbitPage.vue -->
<template>
  <article-layout>

    <template #articleHeader>
      <h1>I am the header</h1>
    </template>

    <template #articleContent>
      <p>I am the content</p>
    </template>

    <template #articleFooter>
      <footer>I am the footer</footer>
    </template>

    <template #side>
      <aside>I am the side stuff</aside>
    </template>

    <template #banner>
      <div>I am the banner</div>
    </template>

  </article-layout>
<template>
```
#### 动态指令参数
❝指令参数现在可以接受动态JavaScript表达式 动态参数值应该是字符串，但允许null作为一个明确指示应该删除绑定的特殊值，那将会很方便。任何其他非字符串值都可能出错，并会触发警告。（仅适用于v-bind和v-on）
<div v-bind:[attr]="attributeName"></div>
//简写
<div :[attr]="attributeName"></div>
这里的 attributeName 会被作为一个JavaScript表达式进行动态求值，求得的值将会作为最终的参数来使用。例如，如果你的 Vue 实例有一个 data 属性 attributeName，其值为 href，那么这个绑定将等价于 v-bind:href

同样地，你可以使用动态参数为一个动态的事件名绑定处理函数：

<button v-on:[eventName]="handler"></button>
//简写
<button @[eventName]="handler"></button>
当 eventName 的值为 focus 时，v-on:[eventName] 将等价于 v-on:focus。

同样可以适用于插槽绑定:
``` javascript
<my-component>
<template v-slot:[slotName]>
Dynamic slot name
</template>
</my-component>
//简写
<foo>
<template #[name]>
Default slot
</template>
</foo>
```
动态参数预期会求出一个字符串，异常情况下值为 null。这个特殊的 null 值可以被显性地用于移除绑定。任何其它非字符串类型的值都将会触发一个警告。
<!-- 这会触发一个编译警告 且 无效 -->
<a v-bind:['foo' + bar]="value"> ... </a>
变通的办法是使用没有空格或引号的表达式，或用计算属性替代这种复杂表达式。
hook那些事

``` javascript
<script>
  export default {
    mounted() {
      this.timer = setInterval(() => { ... }, 1000);
    },
    beforeDestroy() {
      clearInterval(this.timer);
    }
  };
</script>
```

``` javascript
<script>
  export default {
    mounted() {
      const timer = setInterval(() => { ... }, 1000);
      this.$once('hook:beforeDestroy', () => clearInterval(timer);)
    }
  };
</script>
```

``` javascript
<v-chart
    @hook:mounted="loading = false"
    @hook:beforeUpdated="loading = true"
    @hook:updated="loading = false"
    :data="data"
/>
```
实现了对子组件生命周期的监听。对任意的组件都有效果，包括引入的第三方组件
#### vue中的$props、$attrs和$listeners(可用来二次封装组件)

``` javascript
<input v-bind="$props">
```
``` javascript
<input
   v-bind="$attrs"
   :value="value"
   @focus=$emit('focus', $event)"
   @input="$emit('input', $event.target.value)"
>
```
#### jsx模板组件

``` javascript
<!DOCTYPE html>
<html lang="en">

<body>
    <div id="app">
        <child :status="status"></child>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        Vue.component('child', {
            props: {
                status: {
                    type: Number,
                    required: true
                }
            },
            render(createElement) {
                const innerHTML = ['未开始', '进行中', '可领取', '已领取'][this.status]
                return createElement('button', {
                    class: {
                        active: this.status
                    },
                    attrs: {
                        id: 'btn'
                    },
                    domProps: {
                        innerHTML
                    },
                    on: {
                        click: () => console.log(this.status)
                    }
                })
            }
        })
        var app = new Vue({
            el: '#app',
            data: {
                status: 0
            }
        })
    </script>
</body>

</html>
```