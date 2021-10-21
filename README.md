node 版本更新：
1、首先需要清除原来node的 cache：
sudo npm cache clean -f

2、安装n模块
sudo npm install -g n 

安装成功我们就用使用n命令了

3、查看一下node所有的版本信息
 npm view node versions

我们可以选择任意一个版本:
sudo n 15.0.0

等安装完成 看一下版本即可：node -v




vue3 script setup 定稿
vue script setup 已经官宣定稿。本文主要翻译了来自 0040-script-setup 的内容。

摘要
在单文件组件（SFC）中引入一个新的 <script> 类型 setup。它向模板公开了所有的顶层绑定。

基础示例
复制代码
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
HTML
<script setup>
  //imported components are also directly usable in template
  import Foo from './Foo.vue'
  import { ref } from 'vue'

  //write Composition API code just like in a normal setup ()
  //but no need to manually return everything
  const count = ref(0)
  const inc = () => {
    count.value++
  }
</script>

<template>
  <Foo :count="count" @click="inc" />
</template>
编译输出
声明 Props 和 Emits

复制代码
1
2
3
4
5
6
7
8
HTML
<script setup>
  //expects props options
  const props = defineProps({
    foo: String,
  })
  //expects emits options
  const emit = defineEmits(['update', 'delete'])
</script>
动机
这个提案的主要目标是通过直接向模板公开 <script setup> 的上下文，减少在单文件组件（SFC）中使用 Composition API 的繁琐程度。

之前有一个关于 <script setup> 的提案 这里，目前已经实现（但被标记为实验性）。旧的提议选择了导出语法，这样代码就能与未使用的变量配合得很好。

这个提议采取了一个不同的方向，基于我们可以在 eslint-plugin-vue 中提供定制的 linter 规则的前提下。这使我们能够以最简洁的语法为目标。

设计细节
使用 script setup 语法
要使用 script setup 语法，直接在 script 标签中加入 setup 就可以了

复制代码
1
2
3
HTML
<script setup>
  //syntax enabled
</script>
暴露顶级绑定
当使用 <script setup> 时，模板被编译成一个渲染函数，被内联在 setup 函数 scope 内。这意味着任何在 <script setup> 中声明的顶级绑定（top level bindings）（包括变量和导入）都可以直接在模板中使用。

复制代码
1
2
3
4
5
6
7
HTML
<script setup>
  const msg = 'Hello!'
</script>

<template>
  <div>{{ msg }}</div>
</template>
编译输出
注意到模板范围的心智模型与 Options API 的不同是很重要的：当使用 Options API 时，<script> 和模板是通过一个 “渲染上下文对象” 连接的。当我们写代码时，我们总是在考虑 “在上下文中暴露哪些属性”。这自然会导致对 “在上下文中泄露太多的私有逻辑” 的担忧。

然而，当使用 <script setup> 时，心理模型只是一个函数在另一个函数内的模型：内部函数可以访问父范围内的所有东西，而且因为父范围是封闭的，所以没有 "泄漏" 的问题。

使用组件
<script setup> 范围内的值也可以直接用作自定义组件标签名，类似于 JSX 中的工作方式。

复制代码
1
2
3
4
5
6
7
8
9
10
HTML
<script setup>
  import Foo from './Foo.vue'
  import MyComponent from './MyComponent.vue'
</script>

<template>
  <Foo />
  <!-- kebab-case also works -->
  <my-component />
</template>
编译输出
使用动态组件
复制代码
1
2
3
4
5
6
7
8
9
HTML
<script setup>
  import Foo from './Foo.vue'
  import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
编译输出
使用指令
除了一个名为 v-my-dir 的指令会映射到一个名为 vMyDir 的 setup 作用域变量，指令的工作方式与此类似：

复制代码
1
2
3
4
5
6
7
HTML
<script setup>
  import { directive as vClickOutside } from 'v-click-outside'
</script>

<template>
  <div v-click-outside />
</template>
编译输出
之所以需要 v 前缀，是因为全局注册的指令（如 v-focus）很可能与本地声明的同名变量发生冲突。v 前缀使得使用一个变量作为指令的意图更加明确，并减少了意外的 “shadowing”。

声明 props 和 emits
为了在完全的类型推导支持下声明 props 和 emits 等选项，我们可以使用 defineProps 和 defineEmits API，它们在 <script setup> 中自动可用，无需导入。

复制代码
1
2
3
4
5
6
7
8
HTML
<script setup>
  const props = defineProps({
    foo: String,
  })

  const emit = defineEmits(['change', 'delete'])
  //setup code
</script>
编译输出
defineProps 和 defineEmits 根据传递的选项提供正确的类型推理。
defineProps 和 defineEmits 是编译器宏（compiler macros ），只能在 <script setup> 中使用。它们不需要被导入，并且在处理 <script setup> 时被编译掉。
传递给 defineProps 和 defineEmits 的选项将被从 setup 中提升到模块范围。因此，这些选项不能引用在 setup 作用域内声明的局部变量。这样做会导致一个编译错误。然而，它可以引用导入的绑定，因为它们也在模块范围内。
使用 slots 和 attrs
在 <script setup> 中使用 slots 和 attrs 应该是比较少的，因为你可以在模板中直接访问它们，如 $slots 和 $attrs。在罕见的情况下，如果你确实需要它们，请分别使用 useSlots 和 useAttrs 帮助函数（helpers）。

复制代码
1
2
3
4
5
6
HTML
<script setup>
  import { useSlots, useAttrs } from 'vue'

  const slots = useSlots()
  const attrs = useAttrs()
</script>
useSlots 和 useAttrs 是实际的运行时函数，其返回值等价于 setupContext.slots 和 setupContext.attrs。它们也可以在 Composition API 函数中使用。

纯类型的 props 或 emit 声明
props 和 emits 也可以使用 TypeScript 语法来声明，方法是向 defineProps 或 defineEmits 传递一个字面类型参数。

复制代码
1
2
3
4
5
6
7
8
9
TS
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()
defineProps 或 defineEmits 只能使用运行时声明或类型声明。同时使用两者会导致编译错误。
当使用类型声明时，等效的运行时声明会从静态分析中自动生成，以消除双重声明的需要，并仍然确保正确的运行时行为。
在 dev 模式下，编译器将尝试从类型中推断出相应的运行时验证。例如这里的 foo: String 是由 foo: string 类型推断出来的。如果该类型是对导入类型的引用，推断的结果将是 foo: null （等于 any 类型），因为编译器没有外部文件的信息。
在 prod 模式下，编译器将生成数组格式声明以减少包的大小（这里的 props 将被编译成 ['msg']）。
发出的代码仍然是具有有效类型的 TypeScript，它可以被其他工具进一步处理。
截至目前，类型声明参数必须是以下之一，以确保正确的静态分析。
一个类型的字面意义
对同一文件中的接口或类型字的引用
目前不支持复杂类型和从其他文件导入的类型。理论上，将来有可能支持类型导入。

使用类型声明时的默认 props
纯类型的 defineProps 声明的一个缺点是，它没有办法为 props 提供默认值。为了解决这个问题，提供了一个 withDefaults 编译器宏（compiler macros ）。

复制代码
1
2
3
4
5
6
7
TS
interface Props {
  msg?: string
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
})
这将被编译成等效的运行时 props 默认选项。此外，withDefaults 为默认值提供了类型检查，并确保返回的 props 类型对于那些确实有默认值声明的属性来说已经删除了可选标志（optional flags）。

顶级作用域 await
顶层的 await 可以直接在 <script setup> 里面使用。由此产生的 setup () 函数将自动添加 async:

复制代码
1
2
3
HTML
<script setup>
  const post = await fetch(`/api/post/1`).then(r => r.json())
</script>
此外，添加 await 的表达式将被自动编译成一种格式，保留了当前组件实例上下文：

编译输出
暴露组件的公共接口
在传统的 Vue 组件中，所有暴露在模板上的东西都隐含地暴露在组件实例上，可以被父组件通过模板引用检索到。也就是说，在这一点上，模板的渲染上下文和组件的公共接口（public interface）是一样的。这是有问题的，因为这两个用例并不总是完全一致。事实上，大多数时候，在公共接口方面是过度暴露的。这就是为什么要在 Expose RFC 中讨论一种明确的方式来定义一个组件的强制性公共接口。

通过 <script setup> 模板可以访问声明的变量，因为它被编译成一个函数，从 setup () 函数范围返回。这意味着所有声明的变量实际上都不会被返回：它们被包含在 setup () 的闭包中。因此，一个使用 <script setup> 的组件将被默认关闭。也就是说，它的公共的强制性性接口将是一个空对象，除非绑定被明确地暴露出来。

要在 <script setup> 组件中明确地暴露属性，可以使用 defineExpose 编译器宏（compiler macro）。

复制代码
1
2
3
4
5
6
7
8
9
HTML
<script setup>
  const a = 1
  const b = ref(2)

  defineExpose({
    a,
    b,
  })
</script>
编译输出
当父组件通过模板 refs 获得这个组件的实例时，检索到的实例将是 { a: number, b: number } 的样子。（refs 会像普通实例一样自动解包）。通过编译成的内容和 Expose RFC 中建议的运行时等价。

与普通 script 一起使用
有一些情况下，代码必须在模块范围内执行，例如：。

声明命名的出口
只应执行一次的全局副作用。
在这两种情况下，普通的 <script> 块可以和 <script setup> 一起使用。

复制代码
1
2
3
4
5
6
7
8
9
10
HTML
<script>
  performGlobalSideEffect()

  //this can be imported as `import { named } from './*.vue'`
  export const named = 1
</script>

<script setup>
  let count = 0
</script>
Compile Output
name 的自动推导
Vue 3 SFC 在以下情况下会自动从组件的文件名推断出组件的 name。

开发警告格式化
DevTools 检查
递归的自我引用。例如，一个名为 FooBar.vue 的文件可以在其模板中引用自己为 <FooBar/>。
这比明确的注册 / 导入的组件的优先级低。如果你有一个命名的导入与组件的推断名称相冲突，你可以给它取别名。

复制代码
1
TS
import { FooBar as FooBarChild } from './components'
在大多数情况下，不需要明确的 name 声明。唯一需要的情况是当你需要 <keep-alive> 包含或排除或直接检查组件的选项时，你需要这个名字。

声明额外的选项
<script setup> 语法提供了表达大多数现有 Options API 选项同等功能的能力，只有少数选项除外。

name
inheritAttrs
插件或库所需的自定义选项
如果你需要声明这些选项，请使用单独的普通 <script> 块，并使用导出默认值。

复制代码
1
2
3
4
5
6
7
8
9
10
11
HTML
<script>
  export default {
    name: 'CustomName',
    inheritAttrs: false,
    customOptions: {},
  }
</script>

<script setup>
  //script setup logic
</script>
使用限制
由于模块执行语义的不同，<script setup> 内的代码依赖于 SFC 的上下文。当移入外部的.js 或.ts 文件时，可能会导致开发人员和工具的混淆。因此，<script setup> 不能与 src 属性一起使用。

缺陷
工具的兼容性
这种新的范围模型将需要在两个方面进行工具调整。

集成开发环境需要为这个新的 <script setup> 模型提供专门的处理，以便提供模板表达式类型检查 / 道具验证等。

截至目前，Volar 已经在 VSCode 中提供了对这个 RFC 的全面支持，包括所有 TypeScript 相关的功能。它的内部结构也被实现为一个语言服务器，理论上可以在其他 IDE 中使用。

ESLint 规则如 no-unused-vars。我们在 eslint-plugin-vue 中需要一个替换规则，将 <script setup> 和 <template> 表达式都考虑在内。

采用策略
<script setup> 是可选的。现有的 SFC 使用不受影响。

未解决的问题
纯类型的 props/emits 声明目前不支持使用外部导入的类型。这在跨多个组件重复使用基本道具类型定义时非常有用。
在 Volar 的支持下，类型推理已经可以正常工作了，限制纯粹在于 @vue/compiler-sfc 需要知道 props 的键值，以便生成正确的等效运行时声明。

这在技术上是可行的，如果我们实现了跟踪类型导入、读取和解析导入 source 的逻辑。然而，这更像是一个实现范围的问题，并不从根本上影响 RFC 设计的行为方式。

附录
下面的章节仅适用于需要在各自的 SFC 工具集成中支持 <script setup> 的工具作者。

Transform API
@vue/compiler-sfc 包暴露了用于处理 <script setup> 的 compileScript 方法。

复制代码
1
2
3
4
5
6
7
8
9
TS
import { parse, compileScript } from '@vue/compiler-sfc'

const descriptor = parse(`...`)

if (descriptor.script || descriptor.scriptSetup) {
  const result = compileScript(descriptor) //returns SFCScriptBlock
  console.log(result.code)
  console.log(result.bindings) //see next section
}
编译时需要提供整个描述符（the entire descriptor ），产生的代码将包括来自 <script setup> 和普通 <script>（如果存在）中的代码。上层工具（如 vite 或 vue-loader）有责任对编译后的输出进行正确组装。

内联与非内联模式
在开发过程中，<script setup> 仍然编译为返回的对象，而不是内联渲染函数，原因有二：

Devtools 检查
模板热重载（HMR）
内联模板模式只在生产中使用，可以通过 inlineTemplate 选项启用。

复制代码
1
TS
compileScript(descriptor, { inlineTemplate: true })
在内联模式下，一些绑定（例如来自 ref () 调用的返回值）需要用 unref 进行包装。

复制代码
1
2
3
4
5
6
7
8
9
TS
export default {
  setup() {
    const msg = ref('hello')

    return function render() {
      return h('div', unref(msg))
    }
  },
}
编译器会执行一些启发式方法来尽可能地避免这种情况。例如，带有字面初始值的函数声明和常量声明将不会被 unref 包裹。

模板绑定优化
由 compiledScript 返回的 SFCScriptBlock 也暴露了一个 bindings 对象，这是在编译期间收集的导出的绑定元数据。例如，给定以下 <script setup> 。

复制代码
1
2
3
4
5
6
7
HTML
<script setup="props">
  export const foo = 1

  export default {
    props: ['bar'],
  }
</script>
bindings 对象将是。

复制代码
1
2
3
4
TS
{
  foo: 'setup-const',
  bar: 'props'
}
然后这个对象可以被传递给模板编译器。

复制代码
1
2
3
4
5
TS
import { compile } from '@vue/compiler-dom'

compile(template, {
  bindingMetadata: bindings,
})
有了可用的绑定元数据，模板编译器可以生成代码，直接从相应的源码访问模板变量，而不必通过渲染上下文代理。

复制代码
1
HTML
<div>{{ foo + bar }}</div>
复制代码
1
2
3
4
5
6
7
8
9
10
11
TS
//code generated without bindingMetadata
//here _ctx is a Proxy object that dynamically dispatches property access
function render(_ctx) {
  return createVNode('div', null, _ctx.foo + _ctx.bar)
}

//code generated with bindingMetadata
//bypasses the render context proxy
function render(_ctx, _cache, $setup, $props, $data) {
  return createVNode('div', null, $setup.foo + $props.bar)
}
绑定信息也被用于内联模板模式，以生成更有效的代码。

实践
最近，我使用 script setup 语法构建了一个应用 TinyTab —— 一个专注于搜索的新标签页浏览器插件。

🌏 多语言
🎨 切换主题风格
🍍 自定义背景图
🌗 在深色和浅色模式之间切换或跟随系统设置
⛔ 自定义搜索后缀（过滤规则等）
🔍 设置任何你想要的搜索引擎为默认
📦 几个开箱即用的引擎集成
🎭 通过自定义前缀快速切换搜索引擎
🌌 自定义任何支持 OpenSearch 的搜索引擎
🍉 导出与导入任何配置细节
如果你有兴趣了解 vue3 script setup 语法，或许可以查看它。

参考资料
https://sfc.vuejs.org
https://github.com/vuejs/rfcs/blob/master/active-rfcs/0040-script-setup.md
https://github.com/vuejs/rfcs/pull/227#issuecomment-870105222
https://github.com/vuejs/rfcs/pull/227
