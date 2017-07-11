---
layout: post
title: "理解ES6 Promise"
date: 2017-07-01 19:20:31 +0800
comments: true
categories:
---

本文的内容来自于 ES6-Promise-Workshop： https://github.com/benweizhu/es6-promise-workshop

## 什么是Promise？ Promise用来做什么？

延迟操作？网络请求？回调函数？它们统称为“异步操作”。

* User interaction(mouse, keyboard, etc)
* AJAX
* Timers ...

## 为什么大家觉得刚开始写Promise会不太习惯？

因为：

习惯了jQuery的回调
```javascript
$.ajax({
  url: '/user',
  data: { id: 1 },
  success: function (data) {
    console.log(data)
  },
  dataType: 'json'
});
```

习惯同步的Get方法
```java
//Java
User user = userService.getUser(1);
user.getUsername();
```

当有一天

AngularJS通过Service返回一个Promise的时候，我们仍然将Service命名为UserService，但此时返回是一个Promise，而不是User本身。

```javascript
var user = userService.getUser(1);
user.username;
// undefined
```

## Promise是一个JavaScript对象

JavaScript对象创建的方法有两个：字面量和new关键字

## ES6 Promise语法

通过new关键字创建一个Promise，并传递一个函数作为参数
```javascript
var promise = new Promise(function (resolve, reject) {
  // 业务代码
});
```
**Promise中业务代码的执行有两个结果：成功（resolve）或者 失败（reject）**

成功调用resolve
```javascript
var promise = new Promise(function (resolve) {
  resolve(42); // pass 42 to then cb
});
promise.then(function (value) {
  console.log(value);
});
```

失败调用reject
```javascript
var promise = new Promise(function (resolve, reject) {
  reject(new Error('error')); // pass Error obj to catch cb
});
promise.catch(function (error) {
  console.log(error);
});
var promise = new Promise(function (resolve, reject) {
  reject(new Error('error')); // pass Error obj to catch cb
});
promise.then(resolveCb, function(error){
  console.log(error);
});
```

这是常见的Promise教程顺其自然的语法讲解，resolve会将传入的参数传递给then的回调函数，reject会传递给catch或者then的第二个参数。

## Promise是异步操作

这个时候，我们思考一个问题：我们一直说Promise是解决异步操作的，那么上面的代码中，哪一部分是异步的呢？

先思考下，异步操作中，到底哪一步是异步，比如，ajax调用：代码顺序（同步）执行，发现了一个ajax操作，顺序（同步）执行它，ajax发出一个网络请求，这个网络请求操作交给了浏览器，当网络请求返回，调用对应的callback函数。

真正的异步操作是指这个回调函数，它并没有在JavaScript代码顺序（同步）执行的过程中被调用，而是在晚一些时候才被执行。

那么对于上面的Promise，构造函数传入的函数，是顺序执行的。在这个Promise的传递函数中，没有进行任何的异步操作（比如网络请求），而是顺序执行的，直接调用resolve或者reject将状态设置为成功或者失败。

但是当运行promise.then或者promise.catch，即便当时promise的状态已经是确定的，then和catch里面的函数仍然是异步执行。

## Promise实现网络请求

过去，我们都是使用开源的Promise网络请求工具库，比如Fetch，Axios。今天我们来自己通过ES6 Promoise和XHR实现一个Promise网络请求工具。

代码如下：

```javascript
function fetchData(URL) {
  return new Promise(function (resolve, reject) {
    var req = new XMLHttpRequest();
    req.open('GET', URL, true);
    req.onload = function () {
      if (req.status === 200) {
        resolve(this.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function () {
      reject(new Error(req.statusText));
    };
    req.send();
  });
}

var promise = fetchData('https://raw.githubusercontent.com/benweizhu/es6-promise-workshop/master/data/books.json');

promise.then(function (responseText) {
  document.getElementById('json').innerHTML = responseText;
  console.log(JSON.parse(responseText))
}).catch(function (error) {
  console.log(error)
})
```

其实非常简单，你只要记得在fetchData执行完之后，你需要一个promise，那么fetchData中就需要通过new关键字创建并返回，剩下的就是将XHR的操作放在传入的构造函数中。

## Promise Chain

先看代码：
```javascript
function increment(value) { return value + 1; }
function output(value) { console.log(value); }
/**  1 + 1 = 2 **/

var promise = Promise.resolve(1);

promise
  .then(increment)
  .then(output);
```
Promise.resolve(1)是Promise提供的快速创建一个Promise的方法。

这里，我们通过代码反向推导，promise可以调用then或者catch方法，当我们看到then方法后面可以继续调用then方法时，就可以明白，then方法也返回了一个promise，这个promise的then方法中的函数接收到的参数是上一个then方法中的函数return的结果。

假设现在我们来实现( 1 + 1 ) * 2 = 4

```javascript
function doubleUp(value) { return value * 2; }
function increment(value) { return value + 1; }
function output(value) { console.log(value); }
/** ( 1 + 1 ) * 2 = 4 **/

var promise = Promise.resolve(1);

promise
  .then(increment)
  .then(doubleUp)
  .then(output);
```

那么，这里很容混淆的时候，以前你可能会认为.then方法之所以可以chain，是因为then的函数中返回了一个promise，但其实不是这个原因。

**那么，如果真的返回了一个promise，结果是什么呢?答案是：**

如果你返回类似于promise的内容，下一个then()则会等待，并仅在promise产生结果（成功/失败）时调用

举个例子：
```javascript
function resolveAfterTime(num, time) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      resolve(num);
    }, time)
  });
}

resolveAfterTime(10, 1000).then(function (value) {
  console.log(value)
  return resolveAfterTime(value + 10, 5000);
}).then(function (value) {
  console.log(value)
});
```
第一个log在1秒后打印，第二个log在5秒后打印。

## 终极作业

链式调用请求书列表中每本书的详细内容，并返回JSON数据

https://raw.githubusercontent.com/benweizhu/es6-promise-workshop/master/data/books.json

```json
[
  {
    "id": 1,
    "name": "《重构 改善既有代码的设计》",
    "price": 100,
    "url": "https://raw.githubusercontent.com/benweizhu/es6-promise-workshop/master/data/refactoring.json"
  },
  {
    "id": 2,
    "name": "《JavaScript编程精粹》",
    "price": 100,
    "url": "https://raw.githubusercontent.com/benweizhu/es6-promise-workshop/master/data/javascript-the-good-parts.json"
  }
]
```

### 这里我们会用到Promise.all

Promise.all: 接收一个promise对象的数组作为参数，当这个数组里的所有promise对象全部变为resolve或reject状态的时候，它才会去调用.then方法。


```javascript
function fetchData(URL) {
  return new Promise(function (resolve, reject) {
    var req = new XMLHttpRequest();
    req.open('GET', URL, true);
    req.onload = function () {
      if (req.status === 200) {
        resolve(req.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function () {
      reject(new Error(req.statusText));
    };
    req.send();
  });
}

fetchData("https://raw.githubusercontent.com/benweizhu/es6-promise-workshop/master/data/books.json")
  .then(function (data) {
    var books = JSON.parse(data);
    var booksPromise = books.map(function (book) {
      return fetchData(book.url);
    });
    return Promise.all(booksPromise);
  })
  .then(function (bookDetailsList) {
    bookDetailsList.forEach(function (bookDetails) {
      var img = document.createElement("img");
      img.src = JSON.parse(bookDetails).imageUrl;
      document.body.appendChild(img);
    });
  })
  .catch(function (error) {
    console.error(error);
  });
```
