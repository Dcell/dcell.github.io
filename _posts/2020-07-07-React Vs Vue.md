---
title: "React Vs Vue"
layout: post
comments: true
---

# React Vs Vue
首先声明，只单纯的花了几周把2个框架的官方文档看了遍和Demo写了下，纯粹个人意见；如果熟悉了任何一个框架，完全没必要去学习另外一个，因为2个都很优秀。
## 如果你不想用组件式开发，选React
```   
    <div id="app">
        {{ message }}
    </div>
    <script>
        var app = new Vue({
            el:'#app',
            data:{
                message:'Test'
            }
        });
    </script>
```

## 如果你想用模版开发，选Vue （JSX Vs 模版）

```
<template>
    <div> {{date}} </div>
</template>

<script>
    export default {
        name: "MyFristCompent",
        data:{
            date:new Date(),
        }
    }
</script>

<style scoped>
    div{
        background-color: red;
    }
</style>
```
Vue：提供了模版的方式，将Html，CSS和Script分离，看起来比较的舒服。
重点：Css样式可以当前组件生效，这个还是比较重要的一个功能。


```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
如果你对JSX很熟，那么选React挺好的
React也支持外联内联样式，在React里面，有些Css样式用JSX写，key值是不一样的，很难受。
整个React组件看起来比较混乱（相比Vue）
## 如果你喜欢简单，选Vue
React里的状态（state）是不可变（immutable）的，因此你不能直接地改变它，而是要用setState API方法：

```
this.setState({ 
    message: this.state.message.split('').reverse().join('') 
});
```
> 其实这个还有一个坑，setState是异步的，不能作为组件的一个属性来存储数据，如果你要setState后立即需要用的message属性，我劝你再写一份属性。


相对React Vue就比较简单：

```
this.message = this.message.split('').reverse().join('');
```
更加厉害的是，Vue中改变状态的操作不仅更加简洁，而且它的重新渲染系统实际上比React的更快更高效。
当然，Vue里面还有更多高级语法，在我看来，作者参考了很多现代语言，比如拦截器，计算属性等等

## 如果你有类似Tab/Navigation，而不想重新渲染，选Vue
vue路由官方支持Keep-alive
react路由暂时不支持，需要有第3方的状态管理/或者其他路由来代替。
## 如果你要构建适配原生App的框架，选React
React Native是一个用于通过Javascript构建移动端原生应用程序的库。 它与React.js相同，只是不使用Web组件，而是使用原生组件。 如果你学过React.js，很快就能上手React Native，反之亦然。

## 如果你想要最大的生态圈，选React
老牌框架，生态比较成熟

## 如果你构建管理大型应用，选React
> 对于管理大型应用中的状态这一话题而言，Vue.js的作者尤雨溪曾说过，（Vue的）解决方案适用于小型应用，但对于对于大型应用而言不太适合

但是2个框架都提供了，Redux或Vuex状态管理


-------
## 最后再看下Github的情况
React:12.6W🌟
Issues:close 6840；剩余465

Vue:13.5W🌟
Issues:close 7671；剩余187








