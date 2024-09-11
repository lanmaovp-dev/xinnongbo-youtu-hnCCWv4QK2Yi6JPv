[合集 \- vue3代码修炼秘籍(17\)](https://github.com)[1\.答应我，在vue中不要滥用watch好吗？02\-29](https://github.com/heavenYJJ/p/18045325)[2\.一文搞懂 Vue3 defineModel 双向绑定：告别繁琐代码！02\-04](https://github.com/heavenYJJ/p/18006048)[3\.没有虚拟DOM版本的vue（Vue Vapor）01\-26](https://github.com/heavenYJJ/p/17988599)[4\.有了Composition API后，有些场景或许你不需要pinia了01\-23](https://github.com/heavenYJJ/p/17982518)[5\.你不知道的vue3：使用runWithContext实现在非 setup 期间使用inject01\-17](https://github.com/heavenYJJ/p/17970284)[6\.直接在\*.vue文件（SFC）中使用JSX/TSX渲染函数，真香!01\-12](https://github.com/heavenYJJ/p/17960753)[7\.5分钟搞定vue3函数式弹窗01\-10](https://github.com/heavenYJJ/p/17955954)[8\.看不懂来打我，vue3如何将template编译成render函数04\-12](https://github.com/heavenYJJ/p/18129909)[9\.终于搞懂了！原来 Vue 3 的 generate 是这样生成 render 函数的05\-20](https://github.com/heavenYJJ/p/18200376)[10\.涨见识了！脱离vue项目竟然也可以使用响应式API07\-25](https://github.com/heavenYJJ/p/18321516):[veee加速器](https://liuyunzhuge.com)[11\.70%的人都答错了的面试题，vue3的ref是如何实现响应式的？07\-29](https://github.com/heavenYJJ/p/18328847)[12\.用了组合式 (Composition) API 后代码变得更乱了，怎么办？08\-02](https://github.com/heavenYJJ/p/18337613)[13\.给我5分钟，保证教会你在vue3中动态加载远程组件08\-07](https://github.com/heavenYJJ/p/18346051)[14\.卧槽，牛逼！vue3的组件竟然还能“暂停”渲染！08\-19](https://github.com/heavenYJJ/p/18366122)[15\.牛逼！Vue3\.5的useTemplateRef让ref操作DOM更加丝滑09\-04](https://github.com/heavenYJJ/p/18395547)[16\.这应该是全网最详细的Vue3\.5版本解读09\-05](https://github.com/heavenYJJ/p/18397413)17\.牛逼！在Vue3\.5中仅仅2分钟就能封装一个自动cancel的fetch函数09\-11收起
# 前言


在欧阳的上一篇 [这应该是全网最详细的Vue3\.5版本解读](https://github.com)文章中有不少同学对Vue3\.5新增的`onWatcherCleanup`有点疑惑，这个新增的API好像和`watch API`回调的第三个参数`onCleanup`功能好像重复了。今天这篇文章来讲讲新增的`onWatcherCleanup`函数的使用场景：`封装一个自动cancel的fetch函数`。


关注公众号：【前端欧阳】，给自己一个进阶vue的机会


# watch回调的第三个参数onCleanup


有些同学可能还不清楚`watch`回调的第三个参数`onCleanup`，我们先来看个demo，代码如下：



```
watch(id, (value, oldValue, onCleanup) => {
  console.log("do something");
  onCleanup(() => {
    console.log("cleanup");
  });
});

```

`watch`回调的前两个参数大家应该很熟悉，分别是`value`新的值，`oldValue`旧的值。


第三个参数`onCleanup`大家平时可能用的不多，这是一个回调函数，当`watch`的值改变后或者组件销毁前就会执行`onCleanup`传入的回调。


在上面的demo中就是变量`id`改变时会触发`onCleanup`中的回调，进而`console`打印`"cleanup"`字符串。又或者所在的组件销毁前也会触发`onCleanup`中的回调，进而`console`打印`"cleanup"`字符串。


那我们在`onCleanup`中可以干嘛呢？


答案是可以清理副作用，比如在watch中使用`setInterval`初始化一个定时器。那么我们就可以在`onCleanup`的回调中清理掉定时器，无需去组件的`beforeUnmount`钩子函数去统一清理。


# `onWatcherCleanup`函数


`onWatcherCleanup`函数的作用和`watch`回调的第三个参数`onCleanup`差不多，也是当`watch`的值改变后或者组件销毁前就会执行`onWatcherCleanup`传入的回调。


使用方法也很简单，代码如下：



```
import { watch, onWatcherCleanup } from "vue";

watch(id, () => {
  console.log("do something");
  onWatcherCleanup(() => {
    console.log("cleanup");
  });
});

```

从上面的代码可以看到`onWatcherCleanup`的用法其实和`watch`回调的第三个参数`onCleanup`差不多，区别在于这里的`onWatcherCleanup`是从vue中import导入的。


除了从vue中import导入的区别以外，还有一个区别是`onWatcherCleanup`不光在`watch`中可以使用，在`watchEffect`中同样也可以使用。比如下面这样的：



```
watchEffect(() => {
  console.log("do something in watchEffect", id.value);
  onWatcherCleanup(() => {
    console.log("cleanup watchEffect");
  });
});

```

和前面的例子一样，上面的代码中`id`的值改变后或者组件销毁时也会执行`onWatcherCleanup`函数中的`console.log`打印。


`onWatcherCleanup`函数是从vue中import导入的，那么这意味着`onWatcherCleanup`函数的调用可以写在任意地方，只要最终经过函数的层层调用后还是在`watch`或者`watchEffect`的回调中就可以。


利用上面的这一特点我们可以使用`onWatcherCleanup`做到一些`onCleanup`做不到的事情，比如：封装一个自动`cancel`的`fetch`函数。


# 封装自动cancel的fetch函数


在讲这个之前我们先来了解一下如何`cancel`一个`fetch`函数。


这里涉及到`AbortController`接口，**`AbortController`** 接口表示一个控制器对象，允许你根据需要中止一个或多个 Web 请求。


下面这个是`cancel`取消一个请求的demo，代码如下：



```
const controller = new AbortController();
const res = await fetch(url, {
  ...options,
  signal: controller.signal,
});

setTimeout(() => {
  controller.abort();
}, 500);

```

首先使用`new AbortController()`创建一个控制器对象`controller`。


其中的`controller.signal`返回一个 [`AbortSignal`](https://github.com) 对象实例，可以用它来和异步操作进行通信或者中止这个操作。


在我们这里把`controller.signal`作为`signal`选项直接传给fetch函数就可以了。


最后就是可以使用`controller.abort()`将fetch请求取消掉，在上面的demo中是如果超过500ms请求还没完成，那么就执行`controller.abort()`将fetch请求取消掉。


有了前面的知识铺垫，我们先来看看使用“自动`cancel`的`fetch`函数”的地方，代码如下：



```


<template>
  <p>data is: {{ data }}p>
  <button @click="id++">id++button>
template>

```

在上面的例子中使用`watch`监听了变量`id`，在监听的回调中会使用封装的`myFetch`函数请求接口。


上面的例子大家平时应该经常遇到，如果`id`的值变化很快，但是服务端接口请求需要2秒才能完成，这时我们期望只有最后一次`id`的值改变触发的请求才需要完成，其他请求都cancel取消掉。


如果在`myFetch`请求的过程中组件被销毁了，此时我们也期望能够将请求cancel取消掉。


在Vue3\.5之前想要去实现上面的这两个需求很麻烦，但是有了Vue3\.5的`onWatcherCleanup`函数后就非常容易了。


这个是封装的自动`cancel`的`fetch`函数，`myFetch.ts`文件代码如下：



```
import { getCurrentWatcher, onWatcherCleanup } from "vue";

export default async function myFetch(url: string, options: RequestInit) {
  const controller = new AbortController();
  if (getCurrentWatcher()) {
    onWatcherCleanup(() => {
      controller.abort();
    });
  }

  const res = await fetch(url, {
    ...options,
    signal: controller.signal,
  });

  let json;
  try {
    json = await res.json();
  } catch (error) {
    json = {
      code: 500,
      message: "JSON format error",
    };
  }
  return json;
}

```

由于`onWatcherCleanup`函数是从vue中import导入，那么我们就可以在自己封装的`myFetch`函数中导入和使用他。


在`onWatcherCleanup`函数的回调中我们执行了`controller.abort()`，前面已经讲过了当`watch`或者`watchEffect`的回调执行前或者组件卸载前就会执行里面的`onWatcherCleanup`注册的回调。我们这里的`myFetch`是在`watch`中调用的，当然也会触发里面的`onWatcherCleanup`注册的回调。


在`onWatcherCleanup`的回调中执行了`controller.abort()`，前面我们讲过了执行`controller.abort()`就会将正在请求的fetch函数给cancel取消掉。


就这么简单的就实现了前面的两个需求：


需求一：**如果`id`的值变化很快，但是服务端接口请求需要2秒才能完成，这时我们期望只有最后一次`id`的值改变触发的请求才需要完成，其他请求都cancel取消掉。**下面这个是变量id在短时间内多次修改的gif效果图：
![click](https://img2024.cnblogs.com/blog/1217259/202409/1217259-20240910234101295-162887957.gif)


从上面的gif图可以看到只有最后一个请求是完成了的，其他请求全部被cancel掉。


需求二：**如果在`myFetch`请求的过程中组件被销毁了，此时我们也期望能够将请求cancel取消掉。**下面这个是组件卸载时gif效果图：
![hide](https://img2024.cnblogs.com/blog/1217259/202409/1217259-20240910234117698-332813226.gif)


从上图中可以看到在卸载组件时组件正在从服务端请求数据，此时请求会自动cancel掉。


细心的小伙伴发现了在`myFetch`函数中，`onWatcherCleanup`函数外面套了一个`getCurrentWatcher`的判断，代码如下：



```
import { getCurrentWatcher, onWatcherCleanup } from "vue";

export default async function myFetch(url: string, options: RequestInit) {
  // ...省略
  if (getCurrentWatcher()) {
    onWatcherCleanup(() => {
      controller.abort();
    });
  }
  // ...省略
}

```

当watch或者watchEffect监听的值改变后`onWatcherCleanup`的回调就会触发，所以`onWatcherCleanup`的执行是由其所在的watch或者watchEffect触发的。


如果`onWatcherCleanup`不在watch或者watchEffect的回调中执行，那么当然`onWatcherCleanup`中的回调也永远不会执行。


可能有的小伙伴有疑问，你这里的`onWatcherCleanup`是在`myFetch`中执行的，也没在watch或者watchEffect的回调中执行吖？


答案是`myFetch`函数的执行是在watch中执行的，`myFetch`然后再去执行`onWatcherCleanup`。


而`getCurrentWatcher()`函数就会返回当前**正在执行回调**的watch或者watchEffect，如果当前`myFetch`不是在watch或者watchEffect的回调中执行的，那么`getCurrentWatcher()`函数的返回值就是空，所以这种情况就不需要去执行`onWatcherCleanup`函数了。


最后值得一提的是`onWatcherCleanup`不能在await后面执行，比如下面这样的代码：



```
import { getCurrentWatcher, onWatcherCleanup } from "vue";

export default async function myFetch(url: string, options: RequestInit) {
  const controller = new AbortController();
  const res = await fetch(url, {
    ...options,
    signal: controller.signal,
  });

  let json;
  try {
    json = await res.json();
  } catch (error) {
    json = {
      code: 500,
      message: "JSON format error",
    };
  }
  // ❌ 错误的写法
  if (getCurrentWatcher()) {
    onWatcherCleanup(() => {
      controller.abort();
    });
  }

  return json;
}

```

在上面的代码中我们将`onWatcherCleanup`调用放在了`await fetch()`的后面，这种写法`onWatcherCleanup`注册的**回调是不会执行的**。


为什么在`await`后面的`onWatcherCleanup`注册的回调永远不会执行呢？


答案是js的await相当于注册了一个回调函数去执行await后的代码，当await等待结束后再去执行这个回调函数，从而执行await后的代码。


await以及之前的代码确实是在watch回调中执行的，我们这里的`onWatcherCleanup`就是await后面的代码，await后面的代码是在一个新的回调中执行的，也就是watch“回调中”的“回调中”执行的。


当`onWatcherCleanup`执行时已经不知道当前正在执行的watch回调是谁了，所以`onWatcherCleanup`的回调也没注册上。当watch的变量修改时或者组件卸载时`onWatcherCleanup`注册的回调永远也不会执行。


# 总结


当`watch`或者`watchEffect`监听的变量修改时，以及组件卸载时，会去执行他们回调中使用`onWatcherCleanup`注册的回调函数。并且`onWatcherCleanup`是从vue中import导入的，使得我的可以在任意地方执行`onWatcherCleanup`函数。利用这两个特性我们就可以封装一个自动cancel的fetch函数。


关注公众号：【前端欧阳】，给自己一个进阶vue的机会


![](https://img2024.cnblogs.com/blog/1217259/202406/1217259-20240606112202286-1547217900.jpg)


另外欧阳写了一本开源电子书[vue3编译原理揭秘](https://github.com)，看完这本书可以让你对vue编译的认知有质的提升。这本书初、中级前端能看懂，完全免费，只求一个star。


