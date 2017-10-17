### createApplication 函数的返回值
---

若想知道返回值是什么，我们先看看 express.js 中 createApplication 的定义
```javascript
function createApplication() {
  var app = function(req, res, next) {
    app.handle(req, res, next);
  };

  mixin(app, EventEmitter.prototype, false);
  mixin(app, proto, false);

  // expose the prototype that will get set on requests
  app.request = Object.create(req, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  // expose the prototype that will get set on responses
  app.response = Object.create(res, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  app.init();
  return app;
}
```

很明显，返回的是一个叫做 app 的函数，不过经过了一些处理 <br />
经过的处理如下：<br />
    &emsp;&emsp;&emsp;1.app 经过 mixni 处理<br/>
    &emsp;&emsp;&emsp;2.给 a
    pp 添加属性<br />
    &emsp;&emsp;&emsp;3.调用 app.init 方法 <br />


---
<br />

现在， 我们一步步分析这些步骤
#### 1. app 经过 mixni 处理
  mixni 有什么用？ 我们来看看 mixin 的源码<br />
  在 express.js 文件中， 有如下代码
  ```javascript
  var mixin = require('merge-descriptors');
  ```
  所以，mixin变量引用的就是 merge-descriptors 包
<br />
<br />

我们先来了解一下 merge-descriptors 包用到的 api
``` javascript
/*
  1. Object.prototype.hasOwnProperty: 判断调用的对象是否存在传入的属性
*/

var o = { foo: 'hello' };
console.log(o.hasOwnProperty('foo')); // true



/*
  2. Object.getOwnPropertyNames: 对象自身的可枚举和不可枚举属性的名称被返回。
*/

var people = {sex: 'male'};
Object.defineProperty(people, 'name', {
  value: 'manx',
  enumerable: false
});

// Object.keys 只能获得能枚举的属性
console.log(Object.keys(people)); // ['sex']
console.log(Object.getOwnPropertyNames(people)); // [ 'sex', 'name' ]



/*
  3. Object.getOwnPropertyDescriptor: 获得对象的属性描述符
*/

console.log(Object.getOwnPropertyDescriptor(people, 'sex'));
/*
  {
    value: 'male',
    writable: true,
    enumerable: true,
    configurable: true 
  }
*/

console.log(Object.getOwnPropertyDescriptor(people, 'name'));
/*
  { 
    value: 'manx',
    writable: false,
    enumerable: false,
    configurable: false 
  }
*/
```

<br />
<br />


merge-descriptors 包的代码：
```javascript
/*-----------merge-descriptors---start-----------*/
// 我们来看看这个包的代码：
var hasOwnProperty = Object.prototype.hasOwnProperty
function merge(dest, src, redefine) {
  if (!dest) {
    throw new TypeError('argument dest is required')
  }

  if (!src) {
    throw new TypeError('argument src is required')
  }

  // 如果不提供 redefine 默认位 true
  if (redefine === undefined) {
    // Default to true
    redefine = true
  }

  // 遍历源对象的所有属性
  Object.getOwnPropertyNames(src).forEach(function forEachOwnPropertyName(name) {
    // redefine 的作用是： 在目标对象有源对象的属性时候，是否从新给他定义。

    // 这句代码意思是：如果 redefine 为 fasle，而且目标对象有这个属性的话，不从新定义 
    if (!redefine && hasOwnProperty.call(dest, name)) {
      // Skip desriptor
      return
    }

    // 获得当前对象在源对象的属性描述符
    var descriptor = Object.getOwnPropertyDescriptor(src, name)

    // 把当前的属性及在源对象的属性描述符赋给目标对象
    Object.defineProperty(dest, name, descriptor)
  })

  // 返回目标对象
  return dest
}
/*----------------merge-descriptors---end---------------*/
```