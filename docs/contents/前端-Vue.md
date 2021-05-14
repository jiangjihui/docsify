## [**什么是 Vue**](http://www.runoob.com/w3cnote/vue2-start-coding.html)

1 **数据绑定**：比如你改变一个输入框 Input 标签的值，会自动同步更新到页面上其他绑定该输入框的组件的值

2 **组件化：**页面上小到一个按钮都可以是一个单独的文件.vue，这些小组件直接可以像乐高积木一样通过互相引用而组装起来

 

 

## 创建Vue工程

```
npm install vue-cli -g
//安装vue脚手架
cd 目录路径
//在硬盘上找一个文件夹放工程用的，在终端中进入该目录
vue init webpack-simple 工程名字<工程名字不能用中文>
//根据模板创建项目，会有一些初始化的设置，回车默认即可
npm install
//安装项目依赖
cnpm install vue-router vue-resource --save
//安装 vue 路由模块vue-router和网络请求模块vue-resource
npm run dev
//启动项目
```

> Tips：打开工程目录下的 App.vue：template 写 html，script写 js，style写样式

 

 

## **Vue的双向数据绑定**[**原理**](https://segmentfault.com/a/1190000014252365?utm_source=index-hottest)

vue.js 是采用**数据劫持**结合[**发布者**-**订阅者**](https://www.zhihu.com/question/23486749/answer/314072549)模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。

**具体步骤：**

第一步：需要observe的数据对象进行递归遍历，包括子属性对象的属性，都加上 setter和getter这样的话，给这个对象的某个值赋值，就会触发setter，那么就能监听到了数据变化

第二步：compile解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图

第三步：Watcher订阅者是Observer和Compile之间通信的桥梁，主要做的事情是:

1、在自身实例化时往属性订阅器(dep)里面添加自己

2、自身必须有一个update()方法

3、待属性变动dep.notice()通知时，能调用自身的update()方法，并触发Compile中绑定的回调，则功成身退。

第四步：MVVM作为数据绑定的入口，整合Observer、Compile和Watcher三者，通过Observer来监听自己的model数据变化，通过Compile来解析编译模板指令，最终利用Watcher搭起Observer和Compile之间的通信桥梁，达到数据变化 -> 视图更新；视图交互变化(input) -> 数据model变更的双向绑定效果。

 

## **vue生命周期**

Vue 实例从创建到销毁的过程，就是生命周期。也就是从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、卸载等一系列过程，我们称这是 Vue 的生命周期。

**vue生命周期的作用是什么？**

它的生命周期中有多个事件钩子，让我们在控制整个Vue实例的过程时更容易形成好的逻辑。

**vue生命周期总共有几个阶段？**

它可以总共分为8个阶段：创建前/后, 载入前/后,更新前/后,销毁前/销毁后

**第一次页面加载会触发哪几个钩子？**

第一次页面加载时会触发 beforeCreate, created, beforeMount, mounted 这几个钩子

**DOM 渲染在 哪个周期中就已经完成？**

DOM 渲染在 mounted 中就已经完成了

**简单描述每个周期具体适合哪些场景？**

生命周期钩子的一些使用方法： beforecreate : 可以在这加个loading事件，在加载实例时触发 created : 初始化完成时的事件写在这里，如在这结束loading事件，异步请求也适宜在这里调用 mounted : 挂载元素，获取到DOM节点 updated : 如果对数据统一处理，在这里写上相应函数 beforeDestroy : 可以做一个确认停止事件的确认框 nextTick : 更新数据后立即操作dom

arguments是一个伪数组，没有遍历接口，不能遍历

 

 

## **前端框架比较**

**Angular1**

AngularJS的工作原理是:HTML模板将会被浏览器解析到DOM中, DOM结构成为AngularJS编译器的输入。AngularJS将会遍历DOM模板, 来生成相应的NG指令,所有的指令都负责针对view(即HTML中的ng-model)来设置数据绑定。因此, NG框架是在DOM加载完成之后, 才开始起作用的。

**React**

React 的渲染建立在 Virtual DOM 上——一种在内存中描述 DOM 树状态的数据结构。当状态发生变化时，React 重新渲染 Virtual DOM，比较计算之后给真实 DOM 打补丁。

Virtual DOM 提供了函数式的方法描述视图，它不使用数据观察机制，每次更新都会重新渲染整个应用，因此从定义上保证了视图与数据的同步。它也开辟了 JavaScript 同构应用的可能性。

在超大量数据的首屏渲染速度上，React 有一定优势，因为 Vue 的渲染机制启动时候要做的工作比较多，而且 React 支持服务端渲染。

React 的 Virtual DOM 也需要优化。复杂的应用里可以选择 1. 手动添加 shouldComponentUpdate 来避免不需要的 vdom re-render；2. Components 尽可能都用 pureRenderMixin，然后采用 Flux 结构 + Immutable.js。其实也不是那么简单的。相比之下，Vue 由于采用依赖追踪，默认就是优化状态：动了多少数据，就触发多少更新，不多也不少。

React 和 Angular 2 都有服务端渲染和原生渲染的功能。

**Vue.js** 

React 和 Vue 有许多相似之处，它们都有：

使用 Virtual DOM（Vue2.0）

提供了响应式(reactive)和可组合的视图组件(composable view component)。

将注意力集中保持在核心库，同时也关注路由和负责处理全局状态管理的辅助库。

 

 

## 与其他框架的区别

**1.与AngularJS的区别**

相同点：

都支持指令：内置指令和自定义指令。

都支持过滤器：内置过滤器和自定义过滤器。

都支持双向数据绑定。

都不支持低端浏览器。

不同点：

1.AngularJS的学习成本高，比如增加了Dependency Injection特性，而Vue.js本身提供的API都比较简单、直观。

2.在性能上，AngularJS依赖对数据做脏检查，所以Watcher越多越慢。

Vue.js使用基于依赖追踪的观察并且使用异步队列更新。所有的数据都是独立触发的。

对于庞大的应用来说，这个优化差异还是比较明显的。

**2.与React的区别**

相同点：

React采用特殊的JSX语法，Vue.js在组件开发中也推崇编写.vue特殊文件格式，对文件内容都有一些约定，两者都需要编译后使用。

中心思想相同：一切都是组件，组件实例之间可以嵌套。

都提供合理的钩子函数，可以让开发者定制化地去处理需求。

都不内置列数AJAX，Route等功能到核心包，而是以插件的方式加载。

在组件开发中都支持mixins的特性。

不同点：

Vue.js在模板中提供了指令，过滤器等，可以非常方便，快捷地操作DOM。

 

## 模块

模块（NodeJS之node-modules文件）

在node.js中模块与文件是一一对应的，也就是说一个node.js文件就是一个模块，文件内容可能是我们封装好的一些JavaScript方法、JSON数据、编译过的C/C++拓展等，在关于node.js的误会提到过node.js的架构。其中http、fs、net等都是node.js提供的核心模块，使用C/C++实现，外部用JavaScript封装。

怎么使外部访问这个module，我们知道客户端的JavaScript使用script标签引入JavaScript文件就可以访问其内容了，但这样带了的弊端很多，最大的就是作用域相同，产生冲突问题，以至于前端大师们想出了立即执行函数等方式，利用闭包解决。node.js使用exports和require对象来解决对外提供接口和引用模块的问题。

test.js

```
var Student = function(){
    var name = '';
this.setName = function(n){
        name=n;
    }; 
this.printName = function(){
        console.log(name)    ;
    };
};
module.exports=Student;
```

使用：

```
// 这样我们的require语句就可以优雅一些了
var Student=require('./test');
```



很神奇的样子，不是说好的exports是模块公开的接口嘛，那么module.exports是什么东西？

**module.exports与exports**

事实的情况是酱紫的，其实module.exports才是模块公开的接口，每个模块都会自动创建一个module对象，对象有一个modules的属性，初始值是个空对象{}，module的公开接口就是这个属性——module.exports。既然如此那和exports对象有毛线关系啊！为什么我们也可以通过exports对象来公开接口呢？

为了方便，模块中会有一个exports对象，和module.exports**指向同一个变量**，所以我们修改exports对象的时候也会修改module.exports对象，这样我们就明白网上盛传的module.exports对象不为空的时候exports对象就自动忽略是怎么回事儿了，因为module.exports通过赋值方式已经和exports对象指向的变量不同了，exports对象怎么改和module.exports对象没关系了。

**require搜索module方式**

node.js中模块有两种类型：核心模块和文件模块，核心模块直接使用名称获取，比如最长用的http模块：

var http=require('http');

在上面例子中我们使用了相对路径 './test'来获取自定义文件模块，那么node.js有几种搜索加载模块方式呢？

核心模块优先级最高，直接使用名字加载，在有命名冲突的时候首先加载核心模块

文件模块只能按照路径加载（可以省略默认的.js拓展名，不是的话需要显示声明书写）：1绝对路径 2相对路径

**一次加载**

无论调用多少次require，对于同一模块node.js只会加载一次，引用多次获取的仍是相同的实例，看个例子：

**test.js**

```typescript
var name='';
function setName(n){
    name=n;
} 
function printName(){
    console.log(name);
}
exports.setName=setName;
exports.printName=printName;
```

**index.js**

```
var test1=require('./test'),
    test2=require('./test');
test1.setName('Byron');
test2.printName();
```

> 执行结果并不是''，而是输出了test1设置的名字，虽然引用两次，但是获取的是一个实例





## 指令

**v-bind 简写**

```
<!-- 完整语法 -->
<a v-bind:href="url"> ... </a>

<!-- 简写 -->
<a :href="url"> ... </a>
```

**v-on 简写**

```
<!-- 完整语法 -->
<a v-on:click="doSomething"> ... </a>

<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

**v-model**

可以通过使用 v-model 指令，在表单 input 和 textarea 元素上创建双向数据绑定。v-model 指令可以根据 input 的 type 类型，自动地以正确的方式更新元素。虽然略显神奇，然而本质上 v-model 不过是「通过监听用户的 input 事件来更新数据」的语法糖，以及对一些边界情况做特殊处理。

v-model 会忽略所有表单元素中 value, checked 或 selected 属性上初始设置的值，而总是将 Vue 实例中的 data 作为真实数据来源。因此你应该在 JavaScript 端的组件 data 选项中声明这些初始值，而不是 HTML 端。





## [单向数据流](https://vuefe.cn/v2/guide/components.html#单向数据流-One-Way-Data-Flow)

所有的 props 都是在子组件属性和父组件属性之间绑定的，按照**自上而下单向流动**方式构成：当父组件属性更新，数据就会向下流动到子组件，但是反过来却并非如此。这种机制可以防止子组件意外地修改了父组件的状态，会造成应用程序的数据流动变得难于理解。

此外，每次父组件更新时，子组件中所有的 props 都会更新为最新值。也就是说，你不应该试图在子组件内部修改 prop。如果你这么做，Vue 就会在控制台给出警告。





## methods和computed的联系和区别

我们可以将同一函数定义为一个 method 或者一个计算属性。对于最终的结果，两种方式确实是相同的。

不同的是**computed**计算属性是基于它们的依赖进行缓存的。计算属性computed只有在它的相关依赖发生改变时才会重新求值。这就意味着只要message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不必再次执行函数。而对于method ，只要发生重新渲染，method 调用总会执行该函数。

**总之：**数据量大，需要缓存的时候用computed；每次确实需要重新加载，不需要缓存时用methods

 



## 组件通信
Props （父->子）
每个组件实例都有自己的孤立隔离作用域。也就是说，不能（也不应该）直接在子组件模板中引用父组件数据。要想在子组件模板中引用父组件数据，可以使用 props 将数据向下传递到子组件。
每个 prop 属性,都可以控制是否从父组件的自定义属性中接收数据。子组件需要使用 props 选项显式声明 props，以便它可以从父组件接收到期望的数据。
使用 v-bind 将 props 属性动态地绑定到父组件中的数据。无论父组件何时更新数据，都可以将数据向下流入到子组件中。（动态绑定）

自定义事件（子->父）
首先使用 $on(eventName) 监听子组件事件，然后使用 $emit(eventName, optionalPayload) 触发该事件，即可实现通知到父组件。
使用emit()触发事件传入参数给父组件的方法进行子组件值到父组件的传递。
子组件是与组件外部环境发生的变化之间完全解耦的。它需要做的就是将自身内部的信息（包括报告给事件触发器(event emitter)的载荷数据）全部通知到父组件中，以防止父组件主动关注子组件内部信息造成耦合。

.sync 修饰符
在某些场景中，我们可能需要对一个 prop 进行「双向绑定」 - 事实上，这个功能在 Vue 1.x 中已经由 .sync 修饰符实现。由于破坏了单向数据流(one-way data flow)的设计，在 2.0 发布时，移除 .sync 修饰符的原因。确实在某些场景中还是需要双向绑定，尤其有助于数据往返于可重用组件。在 2.3.0+ 中，我们为 props 重新引入了 .sync 修饰符，但是这次只是原有语法的语法糖(syntax sugar)包装而成，其背后实现原理是，在组件上自动扩充一个额外的 v-on 监听器。

```js
<comp :foo.sync="bar"></comp>
会被扩充为：
<comp :foo="bar" @update:foo="val => bar = val"></comp>
对于子组件，如果想要更新 foo 的值，则需要显式地触发一个事件，而不是直接修改 prop：
this.$emit('update:foo', newValue)
```





## **Vue内容分发**

vue.js内容分发把组件上下文的内容[注入到组件](https://segmentfault.com/a/1190000007591093)。

标签**<slot>**会把组件使用上下文的内容注入到此标签所占据的位置上。组件分发的概念简单而强大，因为它意味着对一个隔离的组件除了通过属性、事件交互之外，还可以**注入内容**。

一个组件如果需要外部传入简单数据如数字、字符串等等的时候，可以使用property，如果需要传入js表达式或者对象时，可以使用事件，如果希望传入的是HTML标签，那么使用内容分发就再好不过了。所以，尽管内容分发这个概念看起来极为复杂，而实际上可以简化了解为把HTML标签传入组件的一种方法。所以归根结底，内容分发是一种**为组件传递参数**的方法。

 

**命名插槽**

通过slot标签，一股脑的把组件上下文的内容全部注入到组件内的规定位置。vue.js也提供了命名插槽（named slot）的技术，可以把上下文内的内容分成多个有名字的部分，然后插入到组件的不同位置：

```vue
<my-component>
<p slot='slot1'>hi,slot1</p>
<p slot='slot2'>hi,slot2</p>
</my-component>  

Vue.component('my-component', {
  template: `
  <div>
    <slot name='slot1'></slot>
    <slot name='slot2'></slot>
  <div>
```





## **组件API**

在编写组件时，记住是否要复用组件有好处。一次性组件跟其它组件紧密耦合没关系，但是可复用组件应当定义一个清晰的公开接口。

Vue 组件的 API 来自三部分 - **props, events** 和 **slots** ：

- Props 允许外部环境传递数据给组件
- Events 允许组件对外部环境产生副作用(side     effects)
- Slots 允许外部环境将额外的内容组合在组件中。



 

## [**单元素/组件的过渡**](https://vuefe.cn/v2/guide/transitions.html#单元素-组件的过渡)

Vue 提供了 transition 外层包裹容器组件(wrapper component)，可以给下列情形中的任何元素和组件添加进入/离开(enter/leave)过渡

- 条件渲染（使用 v-if）
- 条件展示（使用 v-show）

- 动态组件
- 组件根节点

当插入或删除包含在 transition 组件中的元素时，Vue 将会做以下处理：

1. 自动嗅探目标元素是否使用了     CSS 过渡或动画，如果使用，会在合适的时机添加/移除 CSS 过渡 class。
2. 如果过渡组件设置了 [JavaScript 钩子函数](https://vuefe.cn/v2/guide/transitions.html#JavaScript-Hooks)，这些钩子函数将在合适的时机调用。
3. 如果没有检测到 CSS     过渡/动画，并且也没有设置 JavaScript 钩子函数，插入和/或删除 DOM     的操作会在下一帧中立即执行。（注意：这里的帧是指浏览器逐帧动画机制，和 Vue 的 nextTick 概念不同）

**如何使用：**

1、首先使用<transition name="my">包装需要过渡效果的元素或者组件

2、然后使用过渡类名(.**my**-enter、.**my**-enter-active….)为其添加css过渡效果





## **关注点分离**

一个重要的事情值得注意，**关注点分离不等于文件类型分离。**在现代 UI 开发中，我们已经发现相比于把代码库分离成三个大的层次并将其相互交织起来，把它们划分为松散耦合的组件再将其组合起来更合理一些。在一个组件里，其模板、逻辑和样式是内部耦合的，并且把他们搭配在一起实际上使得组件更加内聚且更可维护。

文件扩展名为 .vue 的 single-file components(单文件组件) 提供了解决方法，并且还可以使用 webpack 或 Browserify 等构建工具。

即便你不喜欢单文件组件，你仍然可以把 JavaScript、CSS 分离成独立的文件然后做到热重载和预编译。

```vue
<!-- my-component.vue -->
<template>
  <div>This will be pre-compiled</div>
</template>
<script src="./my-component.js"></script>
<style src="./my-component.css"></style>
```

