---
title: 浅谈JavaScript中Promise的使用
date: 2017-01-10
tags: 
  - JavaScript
  - ES6
  - Promise
header-img: "learn-promise.png"
---


> Promise是异步编程的一种解决方案。简单来说Promise就是一个容器，里面保存着某个未来才会结束的异步操作的结果。从语法上说，Promise是一个对象，通过它可以获取异步操作的消息。

在谈论Promise之前我们先看一段代码：

```js
step1(function (val) {
    step2(value1, function(val) {
        step3(value2, function(val) {
            step4(value3, function(val) {
                // Do something
            });
        });
    });
});
```

你是否见到过或自己写过这样的代码？
在平日开发中，我们经常会处理一些异步操作，例如发送Ajax请求，setTimeout等，处理这些异步操作我们一般会传递一个回调函数，然而当我们要进行一连串的异步顺序操作的时候，回调代码就自动往右边进行偏离了，就像金字塔那样，相信你已经遇到过了。这就是我们常常说的“金字塔回调”。首先，从视觉看，会有多层的嵌套回调函数，结尾会有大量的花括号和圆括号，例如上述代码。这样的代码不十分优雅，阅读起来也比较费力。我们也无法在内部使用throw new Error()并在外部进行捕获异常。

Promise的出现就是为了主要解决这两个主要问题：它可以让我们已同步的方式编写异步代码，同时我们也可以优雅的捕获错误和异常。它最初起源于社区，现在已经被写入了ES6的规范中，并且主流浏览器都已经开始支持了，所以本文就介绍在代码中如何使用Promise。

### 创建Promise实例

Promise是一个构造函数，使用时我们需要先使用new创建一个Promise实例。

```js
var promise = new Promise(function(resolve, reject) {
  console.log('new promise.');
  setTimeout(function() { // 模拟异步操作
    resolve(Math.random())
  }, 2000);
});

promise.then(function(value) {
  console.log(value);
}, function(error) {
  console.log(error);
});
```

在上面代码中我们可以看到：

  1. 构造函数接受一个函数作为参数，该函数有两个参数 resolve 和 reject ， 它们是两个函数。
  2. resolve 函数的作用是，将Promise对象的状态从 “未完成”变为 “成功”（Pending -> Resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去； reject 函数的作用是，将Promise对象的状态从 “未完成” 变为 “失败”（Pending -> Rejected），在异步操作失败时调用，并将异步操作报出的错误作为参数传递出去。
  3. Promise实例生成以后，可以用 then 方法 指定 Resolved 状态和 Rejected 状态的回调函数。

其中resolve函数的参数除了正常的值以外，还可能是另一个Promise实例，表示异步操作的结果可能是一个值，也有可能是另一个异步操作。这时Promise实例通过then指定的回调会等待另一个Promise实例状态发生变化后才会进行调用。

```js
var promise2 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve(promise); // 传递上一个promise实例
  }, 1000);
});

promise2.then(function(value) {
  console.log(value);
});

```
上述结果是：promise2等待2s之后输出了promise中的值。  
这是promise的状态会传递给promise2, promise的状态决定了promise2的状态。


### Promise实例的状态变化

Promise对象有三种状态

* Pending  （进行中）
* Resolved （已完成）
* Rejected  （已失败）

![promise status](./promise.png)

Promise的状态变化只有上图两条路径，resolve 方法会使Promise对象由Pendding状态变为Resolved状态；reject 方法或者异常会使得Promise对象由pendding状态变为Rejected状态。

对象状态一旦改变，任何时候都能得到这个结果。即状态一旦进入Resolved状态或者Rejected状态， Promise对象便不再出现状态变化，同时我们再添加回调会立即得到结果。这点跟事件不一样，事件是发生后再绑定监听，就监听不到了。

```js
promise.then(function(value) {
  console.log(value);
});
```

### then方法与错误处理

<!-- then可以使异步操作顺序执行 -->

Promise实例生成以后，可以用then为实例添加状态改变时的回调函数。

```js
function getData() {
  return new Promise(function(resolve, reject) {
    var obj = {
      num: 0
    };
    setTimeout(function() { // 模拟异步请求
      obj.num = Math.random();
      resolve(obj);
    }, 1000);
  });
}

getData().then(function(obj) {
  console.log(obj.num); // 0
});

```
getData函数返回一个promise实例，我们接着使用then为它指定了一个Resolved状态的回调函数， 异步请求中传给resolve的值，将作为回调函数中的参数。当异步请求成功之后，回调函数变会执行输出对应的值。  

假设异步请求失败了怎么办？ then其实还可以指定第二个可选的参数，即Rejected状态的回调函数。

```js
getData().then(function(obj) {
  console.log(obj.num); // 0
}, function(error) {
  console.log(error);
});
```

在上述例子中，异步请求成功后，第一个回调函数会执行，如果失败了，第二个回调函数便会执行。其实我们还可以使用catch指定错误时的回调，catch调用其实等同于使用then(undefined, function) 。

```js
getData().then(function(obj) {
  console.log(obj.num);
}).catch(e) {
  console.log(e);
};
```

### 链式调用

我们已经知道了，使用then可以为promise实例添加状态改变的回调函数。其实，我们还可以使用then进行链式调用来指定一组按照顺序执行的回调函数，因为then的调用总是会返回promise的一个新的实例。其中后一个promise实例会依赖上一个promise实例的状态，如果上一个promise实例状态是Rejected,则后面的promise实例状态也是Rejected。如果前一个回调函数返回的是一个promise对象（有异步操作），这时后一个回调函数会等待该promise对象状态发生变化时，才会进行调用。

```js
// return normal value
getData().then(function(obj) {
  console.log(obj.num);
  return obj.num;
}).then(function(value) {
  console.log(value); // obj.num
});

// return another promise
getData().then(function(obj) {
  console.log(obj.num);
  return getData();
}).then(function(value) {
  console.log(value); // new obj
});
```

使用then进行链式调用的时候同样可以使用catch来进行异常捕获。catch的功能更强大，书写方式也更流畅。Promise对象的错误具有冒泡性质，会一直向后传递，直到捕获为止。错误总是会被下一个catch语句捕获。一般来说，不要在then方法里面定义Reject状态的回调函数，总是使用catch捕获错误。

```js
getData().then(function(val) {
  // ...
}).then(function(val) {
  // ...
}).catch(function(e) {
  // 处理前面三个promise产生的错误
  console.log(e);
});
```


### Promise与循环

前面提到了Promise可以指定一组异步操作顺序执行，那如果我们需要等待一组异步操作之后结束之后再执行呢？ Promise提供了一个很方便的方法Promise.all。

```js
getData().then(function(arr) {
  var tasks = [];
  arr.forEach(function(item) {
    tasks.push(remove(item)); // remove为异步操作，返回promise实例
  });
  return Promise.all(tasks);
})
```

Promise.all方法用于将多个Promise实例，包装成一个新的Promise实例。  
Promise.all方法接受一个promise实例数组作为参数（可以不是数组，但需要具有iterator接口），
如果元素不是Promise实例，就会先调用Promise.resolve方法，将参数转为Promise实例，再进一步处理。
Promise.all方法返回的promise实例状态分为两种情况：
* 实例数组中所有实例的状态都变成Resolved， Promise.all返回的实例才会变成Resolved， 并将Promise实例数组的所有返回值组成一个数组，传递给回调函数。
* 实例数组中某个实例变为了Rejected状态，Promise.all返回的实例会立即变为Rejected状态。并将第一个Rejected的实例的返回值传递给回调函数。

```js
var promises = [1,2,3,4].map(function(id) {
  return getData(id);
});

Promise.all(promises).then(function(valArr) {
  // ...
}).catch(function(error) {
  // ...
});
```
**Promise.race** 方法跟Promise.all方法差不多。唯一的区别在于该方法返回的Promise实例并不会等待所有Proimse都跑完，而是只要有一个Promise实例改变状态，它就跟着改变状态。并使用第一个改变状态实例的返回值作为返回值。

### 兼容性


![promise 兼容性](./promise-desc.png)


![promise 兼容性](./promise-mobile.png)

Promise已经被写入ES6规范中。在上图中可以看到，各大浏览器（除IE外）都已开始支持原生Promise的使用。但是在低版本浏览器和运行环境中，并不支持Promise对象。要在这些环境中使用Promise，则需要借助一些兼容Promise的类库。ES6中的Promise规范来源于Promises/A+社区，因此，在选择类库时应该考虑对Promise/A+兼容性。Promise的Polyfill类库有很多，笔者经常使用的有(供参考)：

[es6-promise](https://github.com/stefanpenner/es6-promise)

[bluebird](https://github.com/petkaantonov/bluebird)


### 参考资料

[Promises/A+](https://promisesaplus.com/)  
