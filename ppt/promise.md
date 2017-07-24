---
title: es6 promise
separator: <!--s-->
verticalSeparator: <!--v-->
revealOptions:
    transition: 'fade'
---

## Promise 对象

* [Promise 的含义](#/1)
* [基本用法](#/2)
* [Promise.prototype.then()](#/3)
* [Promise.prototype.catch()](#/4)
* [Promise.all()](#/5)
* [Promise.race()](#/6)
* Promise.resolve()
* Promise.reject()
* 两个有用的附加方法
* 应用
<!--s-->
##  Promise 的含义
* Promise 是一个对象
* Promise 状态：Pending、Resolved（Fulfilled）和Rejected
* Promise 建立之后无法取消,立即执行
<!--s-->
# 基本用法
<!--v-->
* Promise对象是一个构造函数，用来生成Promise实例
~~~
var promise = new Promise(function(resolve, reject) {
  // ... some code
    if (/* 异步操作成功 */){
        resolve(value);
    } else {
        reject(error);
    }
});
~~~
<!--v-->
* 创建promise对象

~~~
function asyncFunction() {
   return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve('Async Hello world');
        }, 16);
    });
};
~~~
<!--v-->
Promise实例生成以后
~~~
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
~~~

~~~
asyncFunction().then(function (value) {
    console.log(value);    // => 'Async Hello world'
}).catch(function (error) {
    console.log(error);
});
~~~
+ 用then法分别指定Resolved状态和Reject状态的回调函数 
+ 第二个函数是可选的，不一定要提供
<!--v-->
Promise 新建后就会立即执行。
~~~
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('Resolved.');
});

console.log('Hi!');
~~~
<!--v-->
立即执行结果：
~~~
// Promise
// Hi!
// Resolved
~~~
<!--v-->
异步加载图片的例子：
~~~
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    var image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
~~~
<!--s-->
## Promise.prototype.then()
[返回](#/)
<!--v-->
* then方法的第一个参数是Resolved状态的回调函数
* 第二个参数（可选）是Rejected状态的回调函数
* then方法返回的是一个新的Promise实例
<!--v-->
链式写法:then方法后面再调用另一个then方法
~~~
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});
~~~
* 第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。
采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。
<!--v-->
~~~
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("Resolved: ", comments);
}, function funcB(err){
  console.log("Rejected: ", err);
});
~~~

~~~
getJSON("/post/1.json").then(
  post => getJSON(post.commentURL)
).then(
  comments => console.log("Resolved: ", comments),
  err => console.log("Rejected: ", err)
);
~~~
<!--v-->

* 第一个then方法指定的回调函数，返回的是另一个Promise对象
* 第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化
* 如果变为Resolved，就调用funcA，如果状态变为Rejected，就调用funcB
<!--s-->
## Promise.prototype.catch()
[返回](#/)
<!--v-->
* Promise.prototype.catch方法 用于指定发生错误时的回调函数。

~~~
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
~~~
<!--v-->
* Promise.prototype.catch方法是.then(null, rejection)的别名

~~~
p.then((val) => console.log('fulfilled:', val))
  .catch((err) => console.log('rejected', err));

// 等同于
p.then((val) => console.log('fulfilled:', val))
  .then(null, (err) => console.log("rejected:", err));
~~~
<!--v-->
* promise抛出一个错误，就被catch方法指定的回调函数捕获

~~~
// 写法一
var promise = new Promise(function(resolve, reject) {
  try {
    throw new Error('test');
  } catch(e) {
    reject(e); //标记状态
  }
});
promise.catch(function(error) {
  console.log(error);
});

// 写法二
var promise = new Promise(function(resolve, reject) {
  reject(new Error('test'));  //标记状态
});
promise.catch(function(error) {
  console.log(error);
});
~~~
<!--v-->

* 未捕获 例子：

~~~
var promise = new Promise(function (resolve, reject) {
  resolve('ok');
  setTimeout(function () { throw new Error('test') }, 0)
});
promise.then(function (value) { console.log(value) });
~~~
Promise 的运行已经结束了，所以这个错误是在 Promise 函数体外抛出的，会冒泡到最外层，成了未捕获的错误
<!--v-->

*  catch方法返回的还是一个 Promise 对象

~~~
var someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing()
.catch(function(error) {
  console.log('oh no', error);
})
.then(function() {
  console.log('carry on');
});
// oh no [ReferenceError: x is not defined]
// carry on
~~~
<!--s-->
## Promise.all()
[返回](#/)
<!--v-->
Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例
~~~
var p = Promise.all([p1, p2, p3]);
~~~
<!--v-->
 生成一个Promise对象的数组

 ~~~
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ... 
}).catch(function(reason){
  // ...    //rejected状态
});
 ~~~

* p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数
* 第一个~~~~注意是第一个
<!--v-->

~~~
const databasePromise = connectDatabase();

const booksPromise = databasePromise
  .then(findAllBooks);

const userPromise = databasePromise
  .then(getCurrentUser);

Promise.all([
  booksPromise,
  userPromise
])
.then(([books, user]) => pickTopRecommentations(books, user));
~~~
<!--s-->
## Promise.race()
[返回](#/)
<!--v-->
Promise.race方法
~~~
var p = Promise.race([p1, p2, p3]);
~~~
p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。
<!--v-->
练习：
~~~
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);
p.then(response => console.log(response));
p.catch(error => console.log(error));
~~~
<!--v-->
指定时间内没有获得结果，就将Promise的状态变为reject，否则变为resolve。

5秒之内fetch方法无法返回结果，变量p的状态就会变为rejected，从而触发catch方法指定的回调函数。
<!--s-->
## Promise.resolve() 
[返回](#/)
<!--v-->

Promise.resolve方法 是 将现有对象转为Promise对象

~~~
var jsPromise = Promise.resolve($.ajax('/whatever.json'));
~~~
Promise.resolve等价于下面的写法

~~~
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
~~~
<!--v-->
Promise.resolve方法的参数分成四种情况
* 参数是一个Promise实例
* 参数是一个thenable对象
* 参数不是具有then方法的对象，或根本就不是对象
* 不带有任何参数
<!--s-->
## Promise.reject() 
[返回](#/)
<!--v-->
Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。

~~~
var p = Promise.reject('出错了');
// 等同于
var p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
~~~
<!--s-->
## Promise.done() 
[返回](#/)
<!--v-->
Promise对象的回调链，不管以then方法或catch方法结尾，要是最后一个方法抛出错误，都有可能无法捕捉到（因为Promise内部的错误不会冒泡到全局）。因此，我们可以提供一个done方法，总是处于回调链的尾端，保证抛出任何可能出现的错误。
<!--v-->

~~~
asyncFunc()
  .then(f1)
  .catch(r1)
  .then(f2)
  .done();
~~~
实现代码:
~~~
Promise.prototype.done = function (onFulfilled, onRejected) {
  this.then(onFulfilled, onRejected)
    .catch(function (reason) {
      // 抛出一个全局错误
      setTimeout(() => { throw reason }, 0);
    });
};
~~~
done都会捕捉到任何可能出现的错误，并向全局抛出。

<!--s-->
## Promise.done() 
[返回](#/)
<!--v-->

finally方法用于指定不管Promise对象最后状态如何，都会执行的操作。它与done方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。

<!--v-->

~~~
server.listen(0)
  .then(function () {
    // run test
  })
  .finally(server.stop);
~~~

实现:

~~~
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
~~~

管前面的Promise是fulfilled还是rejected，都会执行回调函数callback。

