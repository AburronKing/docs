## vue-router时 keep-alive 页面缓存问题解决

对于vue的 keep-alive 组件，先简单介绍一下：

 > ```<keep-alive>``` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 ```<transition>``` 相似，```<keep-alive>``` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。当组件在 ```<keep-alive>``` 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。
 
 >include 和 exclude 属性允许组件有条件地缓存。二者都可以用逗号分隔字符串、正则表达式或一个数组来表示:
 
 ```
 <!-- 逗号分隔字符串 -->
<keep-alive include="a,b">
  <component :is="view"></component>
</keep-alive>

<!-- 正则表达式 (使用 `v-bind`) -->
<keep-alive :include="/a|b/">
  <component :is="view"></component>
</keep-alive>

<!-- Array (use `v-bind`) -->
<keep-alive :include="['a', 'b']">
  <component :is="view"></component>
</keep-alive>
```
>匹配首先检查组件自身的 name 选项，如果 name 选项不可用，则匹配它的局部注册名称（父组件 components 选项的键值）

在项目中，有些页面需要缓存，有些不需要，这就需要使用keep-alive进行相关控制，来达到效果。

一般的使用方法，可能都是下面的这种：

先在路由中配置```meta: { keepAlive: true }```，然后
```
<keep-alive>
  <router-view v-if="$route.meta.keepAlive"></router-view>
</keep-alive>

<router-view v-if="!$route.meta.keepAlive"></router-view>
```

根据keepAlive的值来进行是否缓存判断。这种对于常规的页面缓存是有效的。

但是在项目中，一个页面不会一直被缓存的，有时需要重新渲染。比如，一个列表页（a)、详情页(b)和详情扩展页(c)，a -> b页面，a是缓存页，b需要在每次打开时，重新渲染；b -> c页面，此时b需要被缓存，从c返回时，需要b保持不变。

对于这种需求，目前大部分的解决方案是：通过在路由导航守卫中判断b页面如果是前往c页面，则需要缓存，如果前往a页面，则b不需要缓存。
```
beforeRouteLeave(to, from, next) {
 if(to.path.includes('c')) {
   from.meta.keepAlive = true;
 } else {
   from.meta.keepAlive = false;
 }
   next();
 },
```

这种方式会出现一个问题：

首先，（具体例子，从上到下对于a、b、c），第一次前往c页面时，b确实会被缓存，

![](https://user-gold-cdn.xitu.io/2020/4/30/171c99e4058aaaed?w=472&h=141&f=png&s=16562)

但是我们返回a页面，再前往b页面时，会出现一个新的b缓存页，

![](https://user-gold-cdn.xitu.io/2020/4/30/171c99fe86f10eb8?w=448&h=143&f=png&s=16548)

就算这不影响我们的性能，但是你继续前往c页面时，

![](https://user-gold-cdn.xitu.io/2020/4/30/171c9a0f6fc2654a?w=477&h=137&f=png&s=16551)


我们之前的b被直接销毁，导致返回b页面时，会返回到第一次缓存的页面，而且之后一直都是只回到第一次的页面，

![](https://user-gold-cdn.xitu.io/2020/4/30/171c9a42970c6dd3?w=447&h=114&f=png&s=11867)

这并不是我们想要看到的。

网上搜罗一番，有说可以在回到a页面时，把缓存的b页面手动销毁，vue提供一个```vm.$destroy()```的方法，但是这是不被推荐使用的，我们先试一下：
```
beforeRouteLeave(to, from, next) {
 if(to.path.includes('c')) {
   from.meta.keepAlive = true;
 } else {
   from.meta.keepAlive = false;
   this.$destroy();
 }
   next();
 },
```
继续重复一下上面的操作，发现第一次的页面会被缓存，

![](https://user-gold-cdn.xitu.io/2020/4/30/171c9aa421b0c95e?w=462&h=128&f=png&s=16317)

第二次从a前往b，在前往c之后，b一直会被销毁，无法缓存，有时还会出现没有销毁的情况，导致第一次的缓存一直存在，之后每次还是会销毁b，

![](https://user-gold-cdn.xitu.io/2020/4/30/171c9ae0dfdb4f1d?w=434&h=127&f=png&s=15903)

![](https://user-gold-cdn.xitu.io/2020/4/30/171c9ae576a52ed8?w=463&h=103&f=png&s=12034)

全都不是我们的菜~！！

通过翻读vue文档中的keep-alive的介绍，注意到了它的一个属性：```include```，那就换一种写法，在App.vue中，通过监听路由是否是b到a页面，来判断要不要缓存：
```
<keep-alive :include="keepAlive">
  <router-view></router-view>
</keep-alive>
```
```
export default {
  name: "App",
  data() {
    return {
      keepAlive: ['a', 'b']
    }
  },
  watch: {
    $route(to, from) {
    // 如果是从b到a页面，则不缓存b
      if(from.name === 'b' && to.name === 'a') {
        this.keepAlive = ['a']
      } else {
        this.keepAlive = ['a', 'b']
      }
    }
  }
};
```

大功告成

![](https://user-gold-cdn.xitu.io/2020/4/30/171c9b61055c7ff8?w=476&h=165&f=gif&s=39342)