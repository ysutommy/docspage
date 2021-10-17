### JavaScript重点概念

#### 数据类型

最新的 ECMAScript 标准定义了8种数据类型

- 7种基本数据类型

  - null 

    > 1.  JavaScript 是大小写敏感的，因此 `null` 与 `Null`、`NULL`或变体完全不同
    > 2. ```typeof null === 'object'``` 二进制以000开头都判断为 object，null全为0

  - undefined

  - Boolean

  - Number

  - String

  - BigInt

  - Symbol

- 以及对象（Object）

- 数据类型判断方法

  - ```typeof```只能准确判断基本数据类型

  - `Object.prototype.toString.call(obj)`

  - 判断`constructor`

  - `instanceOf` 

  - **duck type**

    ```js
    function type(obj) {
      return typeof obj !== 'object' ? typeof obj : Object.prototype.toString.call(obj).slice(8, -1).toLowerCase();
    }
    ```

#### 原型/原型链/继承

> 1. String，Array，Number，Function，Object都是function
> 2. `__proto__`和`constructor`是**对象实例**独有的，`prototype`属性是**函数**独有的

1. **原型** JavaScript原型是指为其它对象提供共享属性访问的对象。在创建对象时，每个对象都包含一个隐式应用指向它的原型对象或者null

2. **原型链** 原型也是对象，因此它也有自己的原型，这样构成一个原型链

   - 手写`instanceOf`

     ```js
     function instance_of(L, R) {
       let rp = R.prototype;
       L = L.__proto__;
       while(true) {
         if (L === null) return false;
         if (rp === L) return true;
         L = L.__proto__;
       }
     }
     ```

3. **实现继承的几种方式**

   - 借助构造函数实现继承，缺点是父构造函数的原型链继承不了，若要全部继承，除非将所有属性和方法定义在构造函数中。优点是定义在父构造函数中的引用类型的属性，对于子构造函数的每个实例来说是独立的，并且在子构造函数实例化时，可以给父构造函数传参

     ```js
     function Parent() {
       this.name = 'parent';
     }
     function Child() {
       Parent.call(this);
       this.type = 'type';
     }
     ```

   - 借助原型链实现继承：缺点是继承的引用类型属性是共享的，子构造函数的实例更改会影响其他实例上的这个属性，比如 age 属性

     ```js
     function Parent() {
       this.name = 'parent';
       this.ages = [60];
     }
     function Child() {
       this.type = 'type';
     }
     Child.prototype = new Parent();
     ```

   - 组合方式：缺点是会执行2次父构造函数

     ```js
     function Child() {
       Parent.call(this);
       this.type = 'type';
     }
     Child.prototype = new Parent();

     //组合优化1
     Child.prototype = Object.create(Parent.prototype);
     Child.prototype.constructor = Child;
     ```

   - 寄生组合式继承

     ```js
     //写法1
     function inheritObject(o) {
       function F() {};
       F.prototype = o;
       return new F();
     }

     function inheritPrototype(subClass, superClass) {
       let p = inheritObject(superClass.prototype);
       p.constructor = subClass;
       subClass.prototype = p;
       return p;
     }

     //写法2（ts转js的默认实现）
     function inherit() {
       var extendStatics = Object.setPrototypeOf ||
           ({ __proto__: [] } instanceof Array && function (d, b) { d.__proto__ = b; }) ||
           function (d, b) { for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p]; };
       return function (d, b) {
         extendStatics(d, b);
         function __() { this.constructor = d; }
         d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());
       };
     }
     ```

#### 执行上下文/作用域链/闭包

- **闭包** *闭包是由函数以及声明该函数的词法环境组合而成的，或者说闭包是一个可以自己拥有独立的环境与变量的表达式（通常是函数）*。

  一个函数和对其周围状态（**lexical environment，词法环境**）的引用捆绑在一起（或者说函数被引用包围），这样的组合就是**闭包**（**closure**）。也就是说，闭包让你可以在一个内层函数中访问到其外层函数的作用域。在 JavaScript 中，每当创建一个函数，闭包就会在函数创建的同时被创建出来

- **执行上下文** 包括全局执行上下文、函数执行上下文

  每一个执行上下文，都有三个重要属性：

  - 变量对象(`Variable object，VO`)
  - 作用域链(`Scope chain`)
  - `this`

- **作用域链** 在函数上下文中，查找一个变量foo，如果函数的VO中找到了，就直接使用，否则去它的父级作用域链中`__parent__`找，如果父级中没找到，继续往上找，直到全局上下文中也没找到就报错

#### this/call/apply/bind

- **this是执行上下文环境的一个属性，而不是某个变量对象的属性**


- call/apply 都是将第一个参数替换 fn 函数中的 this 关键字，然后执行 fn 函数

- bind 只是替换 this 关键字，不执行函数

- 原生实现

  **bind**

  ```js
  Function.prototype.bind = function(newThis) {
    let args = [].slice.call(arguments, 1);
    let _this = this;
    return function() {
      return _this.apply(newThis, args.concat([].slice.call(arguments)));
    }
  }
  ```

  **call**

  ```js
  Function.prototype.call = function(ctx, ...args) {
    ctx = ctx || window;
    ctx.fn = this;
    let ret = ctx.fn(...args);
    delete ctx.fn;
    return ret;
  }
  ```

#### new操作符

* new操作符做了哪些事情

  1. 创建一个空的简单JavaScript对象（即`{}`）；
  2. 为步骤1新创建的对象添加属性`__proto__`，将该属性链接至构造函数的原型对象 ；
  3. 将步骤1新创建的对象作为`this`的上下文 ；
  4. 如果该函数没有返回对象，则返回`this`。

* 模拟new操作符

  ```js
  function myNew1(fn) {
    let obj = {};
    obj.__proto__ = fn.prototype;
    let ret = fn.apply(obj, arguments);
    return typeof ret === 'object' ? ret : obj;
  }

  function myNew2(fn) {
      let obj = Object.create(fn.prototype);
      let ret = fn.call(obj, ...arguments);
      return typeof ret === 'object' ? ret : obj;
  }
  ```

#### 箭头函数和普通函数

> **箭头函数表达式**的语法比[函数表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/function)更简洁，并且没有自己的`this`，`arguments`，`super`或`new.target`。箭头函数表达式更适用于那些本来需要匿名函数的地方，并且它不能用作构造函数

- 外形不同
- 箭头函数都是匿名函数
- 箭头函数不能用于构造函数，不能使用`new`
- `this`指向不同。在普通函数中，this总是指向调用它的对象，如果用作构造函数，this指向创建的对象实例
- 箭头函数不具有prototype原型对象

#### Promise

* 4种状态，包括Promise A+规范的三种`pending`、`fulfilled`、`rejected`，`adopted`状态用来表示返回值也是一个promise的情况。如resolve(promise)、在`.then`的回调函数中返回一个promise

- 异步、微任务

- 链式调用，**多个`.then`调用产生任务链（Promise的私有属性`_deferreds`保存下一个需要处理的promise）。按照链条上的先后顺序，前面的执行完成后，后面的才会进入宿主环境（Node、浏览器）的微任务队列，且其自身所传入的回调函数（onFulfilled/onRejected）不一定被执行**。这里有点拗口，详细请见 [Promise详解](c-6-Promise详解)

  > 特殊情况：如果返回值是一个promise，则会被优先处理

- all，race，reject，resolve是静态方法，then，catch，finally，done是原型方法

#### 深拷贝

- 利用`JSON`的`parse`和`stringify`来实现深拷贝

  > undefined，function，symbol会在转换过程中被忽略

- 三方SDK，如lodash。建议使用模块化版本lodash-es，可以按需引入

- 递归

  ```js
  function deepClone(source, map = new WeakMap()) {
    if (typeof source !== 'object' || !source) {
      return source;
    }
    if (map.has(source)) {
      return map.get(source);
    }
    const target = Array.isArray(source) ? [] : {};
    map.set(source, target);
    
    Reflect.ownKeys(source).forEach(key => {
      if (source[key] && typeof source[key] === 'object') {
        target[key] = deepClone(source[key], map);
      } else {
        target[key] = source[key];
      }
    })
    return target;
  }
  ```

- 层序遍历（解决递归爆栈问题）

  ```js
  function levelDeepClone(source) {
    if (typeof source !== 'object' || !source) {
      return source;
    }
    
    const root = Array.isArray(source) ? [] : {};
    const dataList = [
      {data: source, target: root}
    ];
    let node, sre, target;
    const map = new WeakMap();
    
    while(dataList.length > 0) {
      node = dataList.shift();
      sre = node.data;
      target = node.target;
      map.set(sre, target);
      Reflect.ownKeys(sre).forEach(key => {
        if (sre[key] && typeof sre[key] === 'object') {
          if (map.has(sre[key])) {
            target[key] = map.get(sre[key]);
          } else {
            target[key] = Array.isArray(source) ? [] : {};
            dataList.push({data: sre[key], target: target[key]});
          }
        } else {
          target[key] = sre[key];
        }
      })
    }
    
    return root;
  }
  ```

#### 防抖节流的区别及应用

- **防抖** 某一段时间内只执行一次。如搜索联想、window resize

  ```js
  function debounce(fn, delay) {
    return function() {
      let that = this;
      let args = arguments;
      clearTimeout(fn.id);
      fn.id = setTimeout(() => fn.apply(that, args), delay);
    }
  }
  ```

- **节流** 间隔delay触发，如鼠标点击、滚动

  ```js
  function throttle(fn, delay) {
    let _lastTime;
    
    return function() {
      let _this = this;
      let now = Date.now();
      let args = arguments;
      clearTimeout(fn.id);
      
      if (!_lastTime || (_lastTime + delay) <= now) {
        fn.apply(_this, args);
        _lastTime = Date.now();
      } else {
        fn.id = setTimeout(() => {
          fn.apply(_this, args);
          _lastTime = Date.now();
          clearTimeout(fn.id);
          delete fn.id;
        }, delay);
      }
    }
  }
  ```

#### Other