---
layout:     post
title:      初识async与Generator函数
subtitle:   初探async/Generator函数
date:       2018-12-18
author:     Ry
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - ES6
---

### Generator

  ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同。
  
  执行Generator函数会返回一个遍历器对象，也就是说，Generator函数除了状态机，还是一个遍历器对象生成函数，返回的遍历器对象可以遍历Generator函数内部的每一个状态。

  特征：    
    a. function关键字与函数名之间有一个星号（*）；  
    b. 函数体内部使用yield表达式，定义不同的内部状态。
  
  ```js
  function* helloWorldGenerator(){
    yiled 'hello';
    yiled 'world';
    return 'ending';
  }

  var hw = helloWorldGenerator();
  ```
  上面代码中，函数体内有两个yiled表达式，即该函数有三个状态：hello、world和return语句（结束执行） 

  调用该函数后，该函数并不执行，返回的也不是函数的执行结果，而是一个指向内部状态的指针对象

  下一步必须调用next方法，简单来说Generator函数是分段执行的；yield表达式是暂停执行的标记，next方法则可以恢复执行

**yield表达式**

  遍历器对象的next方法的运行逻辑如下：

  （1）遇到yield表达式，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。

  （2）下一次调用next方法时，再继续往下执行，直到遇到下一个yield表达式。

  （3）如果没有再遇到新的yield表达式，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。

  （4）如果该函数没有return语句，则返回的对象的value属性值为undefined。

  需要注意的是，yield表达式后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。


### async函数

  一句话，async函数就是Generator函数的语法糖

  ```js
  // Generator函数写法
  const gen = function* () {
    const f1 = yield readFile('/etc/fstab');
    const f2 = yield readFile('/etc/shells');
    console.log(f1.toString());
    console.log(f2.toString());
  };

  // async函数
  const asyncReadFile = async function () {
    const f1 = await readFile('/etc/fstab');
    const f2 = await readFile('/etc/shells');
    console.log(f1.toString());
    console.log(f2.toString());
  };
  ```

  async函数对 Generator 函数的改进，体现在以下四点：

  （1）内置执行器

  （2）更好的语义

  （3）更广的适用性

  （4）返回值是Promise

**基本用法**

  async函数返回一个promise对象，可以使用then方法添加回调函数。当函数执行时，一旦遇到await就会先等待，等到异步操作完成后在接着执行后面操作。

  使用形式：
  ```js
  // 函数声明
  async function foo() {}

  // 函数表达式
  const foo = async function () {};

  // 对象的方法
  let obj = { async foo() {} };
  obj.foo().then(...)

  // Class 的方法
  class Storage {
    constructor() {
      this.cachePromise = caches.open('avatars');
    }

    async getAvatar(name) {
      const cache = await this.cachePromise;
      return cache.match(`/avatars/${name}.jpg`);
    }
  }

  const storage = new Storage();
  storage.getAvatar('jake').then(…);

  // 箭头函数
  const foo = async () => {};
  ```

**语法**

  * async函数内部return语句返回的值，会成为then方法回调函数的参数
  
  ```js
  async function f() {
    return 'hello world';
  }

  f().then(v => console.log(v)) // 'hello world'
  ```
  
  * async函数内部抛出错误，会导致返回的Promise对象变为reject状态。抛出的错误对象会被catch方法回调函数接收到。
  
  ```js
  async function f() {
    throw new Error('出错了');  // throw 抛出自定义异常
  }

  f().then(
    v => console.log(v),
    e => console.log(e)
  )
  // Error: 出错了
  ```

  * Promise对象的状态变化   
    * async函数返回的Promise对象，必须等到内部所有await命令后的Promise对象执行完，才会发生状态改变，除非遇到return语句或抛出错误。

  * await：   
    * 正常情况下await命令后面是一个Promise对象，返回该对象的结果。如果不是Promise对象，就直接返回对应的值;另一种情况是，await命令后面是一个thenable对象（即定义then方法的对象），那么await会将其等同于Promise对象。

    * 任何一个await语句后的Promise对象变为reject状态，那么整个async函数就会中断执行

    * 有时，我们希望即使前一个异步操作失败，也不要中断后面的异步操作。这时我们可以将第一个await放在try...catch结构里面，这样不管这个异步是否操作成功，第二个await都会执行。

    * 另一种方法是await后面的Promise对象在跟一个catch方法，处理前面可能会出现的错误

  * 错误处理 ---  try...catch     
    * 如果await后面的异步操作出错，那么等同于async函数返回的Promise对象被reject
    ```js
      async function main() {
        try {
          const val1 = await firstStep();
          const val2 = await secondStep(val1);
          const val3 = await thirdStep(val1, val2);

          console.log('Final: ', val3);
        }
        catch (err) {
          console.error(err);
        }
      }
    ```












