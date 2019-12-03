---
layout:     post
title:      Proxy&Reflect初识
subtitle:   控制Object的默认行为
date:       2019-08-12
author:     Ry
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - ES6
    - Proxy
---

### Proxy

  用于修改某些操作的默认行为。

  ```js
  var obj = new Proxy({}, {
    get: function (target, key, receiver) {
      console.log(`getting ${key}!`);
      return Reflect.get(target, key, receiver);
    },
    set: function (target, key, value, receiver) {
      console.log(`setting ${key}!`);
      return Reflect.set(target, key, value, receiver);
    }
  });
  ```
  上面代码对一个空对象架设了一层拦截，重定义了属性的读取（get）和设置（set）行为。

  * 生成Proxy实例：`var proxy = new Proxy(target,handler);`   
    * target：拦截的目标对象    
    * handler：也是一个对象(配置对象)，用于定制拦截行为

  例：拦截读取属性行为
  ```js
  var proxy = new Proxy({},{
    get: function(){
      return 35
    }
  })

  proxy.time // 35
  proxy.name // 35
  ```
  上面代码中，配置对象有一个get方法，用于拦截对目标对象属性的访问请求。get方法的两个参数分别是目标对象和要访问的属性。由于拦截函数（get）总是返回35，所以访问任何属性都得到35。

  注意，要使Proxy生效，必须针对Proxy实例进行操作，而不是针对目标对象进行操作。

  * 13种Proxy支持的拦截方式：

    * get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo']。
  
    * set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
  
    * has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。

    * deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。

    * ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、

    * Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
  
    * getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
  
    * defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
  
    * preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
  
    * getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
  
    * isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。

    * setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

    * apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。

    * construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。

  * Proxy实例中的set方法：
  ```js
    let validator = {
      set: function(obj, prop, value) {
        if (prop === 'age') {
          if (!Number.isInteger(value)) {
            throw new TypeError('The age is not an integer');
          }
          if (value > 200) {
            throw new RangeError('The age seems invalid');
          }
        }

        // 对于满足条件的 age 属性以及其他属性，直接保存
        obj[prop] = value;
      }
    };

    let person = new Proxy({}, validator);

    person.age = 100;

    person.age // 100
    person.age = 'young' // 报错
    person.age = 300 // 报错
  ```
  上面代码中，由于设值存值函数set，任何不符合要求的age属性赋值，都会抛出一个错误，利用set可以实现数据绑定，每当对象发生变化时，会自动更新DOM

    * 利用get和set可以防止一些内部属性被外部读写


  * Proxy.revocable()

    * Proxy.revocable方法返回一个可取消的 Proxy 实例。
    ```js
      let target = {};
      let handler = {};

      let {proxy, revoke} = Proxy.revocable(target, handler);

      proxy.foo = 123;
      proxy.foo // 123

      revoke();
      proxy.foo // TypeError: Revoked
    ```
    * Proxy.revocable方法返回一个对象，该对象的proxy属性是Proxy实例，revoke属性是一个函数，可以取消Proxy实例。上面代码中，当执行revoke函数之后，再访问Proxy实例，就会抛出一个错误。

    * Proxy.revocable的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

  
  * this问题

    * 在Proxy代理的情况下，目标对象内部的this关键字会指向Proxy代理
    ```js
      const target = {
        m: function () {
          console.log(this === proxy);
        }
      };
      const handler = {};

      const proxy = new Proxy(target, handler);

      target.m() // false
      proxy.m()  // true
    ```

### Reflect

* 设计目的：

  1. 将Object对象的一些明显属于语言内部的方法（例如Object.defineProperty），放到Reflect对象上。

  2. 修改某些Object方法的返回结果，让其变得更合理

  3. 让Object操作都变成函数行为

  4. Reflect对象的方法与Proxy对象的方法一一对应，只要Proxy对象的方法，就能在Reflect对象上找到对应的方法。

* 静态方法：

  - Reflect.apply(target, thisArg, args)  

  - Reflect.construct(target, args)   
    - 提供一种不使用new，来调用构造函数的方法

  - Reflect.get(target, name, receiver)   
    -  查找并返回target对象的name属性，如果没有改属性，则返回undefined

  - Reflect.set(target, name, value, receiver)    
    -  设置target对象的name属性等于value

  - Reflect.defineProperty(target, name, desc)    
    -  为对象定义属性

  - Reflect.deleteProperty(target, name)    
    -  等同于delete obj[name]，用于删除对象的属性

  - Reflect.has(target, name)   
    -  对应name in obj里面的in运算符

  - Reflect.ownKeys(target)   

  - Reflect.isExtensible(target)    

  - Reflect.preventExtensions(target)     

  - Reflect.getOwnPropertyDescriptor(target, name)    

  - Reflect.getPrototypeOf(target)
    -  用于读取对象的_proto_属性，对应Object.getPrototypeOf()

  - Reflect.setPrototypeOf(target, prototype)   
    -  用于设置目标对象的原型（prototype），对应Object.setPrototypeOf(obj,newProto)

* 使用Proxy实现观察者模式

  * 观察者模式指的是函数自动观察数据对象，一旦对象有变化，函数就会自动执行
  ```js
    const person = observable({
      name: '张三',
      age: 20
    });

    function print() {
      console.log(`${person.name}, ${person.age}`)
    }

    observe(print);
    person.name = '李四';
    // 输出
    // 李四, 20
  ```

  
