# JavaScript常见手写代码题
## 实现map方法
实现
```javascript
// 方法1：
// thisArg参数就是用来改变回调函数内部this的
Array.prototype.myMap = function (fn, thisArg) {
    // // 首先，检查传递的参数是否正确。
    if (typeof fn !== "function") {
        throw new TypeError(fn + " is not a function");
    }

    // 每次调用此函数时，我们都会创建一个 res 数组, 因为我们不想改变原始数组。
    let res = [];
    for (let i = 0; i < this.length; i++) {
        // 简单处理空项
        this[i] ? res.push(fn.call(thisArg, this[i], i, this)) : res.push(this[i]);
    }
    return res;
};

// 方法2：用reduce实现map方法
Array.prototype.myMap = function(fn, thisArg){
    if (this === null) {
        throw new TypeError("this is null or not defined");
    }
    if (typeof fn !== "function") {
        throw new TypeError(fn + " is not a function");
    }
    var res = [];
    this.reduce(function(pre, cur, index, arr){
            return res.push(fn.call(thisArg, cur, index, arr));
    }, []);
    return res;
}

```
## 实现reduce方法
```javascript
// 实现
Array.prototype.myReduce = function(fn, initValue) {
    // 边界条件判断
    if(typeof fn !== 'function') {
        console.error('this is not a function');
    }
    // 初始值
    let preValue, curValue, curIndex;
    if(typeof initValue === 'undefined') {
        preValue = this[0];
        curValue = this[1];
        curIndex = 1;
    } else {
        preValue = initValue;
        curValue = this[0];
        curIndex = 0;  
    }
    // 遍历
    for (let i = 0; i < this.length; i++) {
        preValue = fn(preValue, this[i], i, this)
    }
    return preValue;
}
```

## 防抖函数
```javascript
// 实现
//参数func：需要防抖的函数
//参数delayTime：延时时长，单位ms
function debounce(func, delayTime) {
    //用闭包路缓存延时器id
    let timer;
    return function (...args) {
        if (timer) {
        	clearTimeout(timer);  //清除-替换，把前浪拍死在沙滩上
        } 
        timer = setTimeout(() => { // 延迟函数就是要晚于上面匿名函数
            func.apply(this, args); // 执行函数
        }, delayTime);
    }
}


// 测试
const task = () => { console.log('run task') }
const debounceTask = debounce(task, 1000)
window.addEventListener('scroll', debounceTask)
```

## 节流函数
```javascript
// 节流函数
function throttle(fn, delay) {
	let timer = null;
	return function(...args) {
		if(timer) {
			return;
		}
		timer = setTimeout(() => {
			fn.apply(this, args);
			timer = null;
		}, delay)
	}
}


// 测试
function print(e) {
	console.log('123', this, e)
}
input.addEventListener('input', throttle(print, 1000));
```
## 深拷贝函数
```javascript
function deepCopy(obj, cache = new WeakMap()) {
  // 数据类型校验
  if (!obj instanceof Object) return obj;
  
  // 防止循环引用，
  if (cache.get(obj)) return cache.get(obj);
  
  // 支持函数
  if (obj instanceof Function) {
    return function () {
      obj.apply(this, arguments);
    }
  }
  // 支持日期
  if (obj instanceof Date) return new Date(obj);
  
  // 支持正则对象
  if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);
  // 还可以增加其他对象，比如：Map, Set等，根据情况判断增加即可，面试点到为止就可以了

  // 数组是 key 为数字素银的特殊对象
  const res = Array.isArray(obj) ? [] : {};
  
  // 缓存 copy 的对象，用于处理循环引用的情况
  cache.set(obj, res);

  Object.keys(obj).forEach((key) => {
    if (obj[key] instanceof Object) {
      res[key] = deepCopy(obj[key], cache);
    } else {
      res[key] = obj[key];
    }
  });
  return res;
}


// 测试
const source = {
  name: 'Jack',
  meta: {
    age: 12,
    birth: new Date('1997-10-10'),
    ary: [1, 2, { a: 1 }],
    say() {
      console.log('Hello');
    }
  }
}
source.source = source
const newObj = deepCopy(source)
console.log(newObj.meta.ary[2] === source.meta.ary[2]);
```
## 实现new函数
```javascript
// 实现
function mynew(Func, ...args) {
    // 1.创建一个新对象
    const obj = {};
    // 2.新对象原型指向构造函数原型对象
    obj.__proto__ = Func.prototype;
    // 3.将构建函数的this指向新对象 (让函数的 this 指向这个对象，执行构造函数的代码（为这个新对象添加属性）)
    let result = Func.apply(obj, args);
    // 4.根据返回值判断
    return result instanceof Object ? result : obj;
}


// 测试
function Person(name, age) {
    this.name = name;
    this.age = age;
}
Person.prototype.say = function () {
    console.log(this.name)
}

let p = mynew(Person, "huihui", 123)
console.log(p) // Person {name: "huihui", age: 123}
p.say() // huihui
```
## 事件总线 | 发布订阅模式
```javascript
// 发布订阅模式
class EventEmitter {
    constructor() {
        // 事件对象，存放订阅的名字和事件
        this.events = {};
    }
    // 订阅事件的方法
    on(eventName, callback) {
       if (!this.events[eventName]) {
           // 注意时数据，一个名字可以订阅多个事件函数
           this.events[eventName] = [callback]
       } else  {
          // 存在则push到指定数组的尾部保存
           this.events[eventName].push(callback)
       }
    }
    // 移除订阅事件
    off(eventName, callback) {
        if (!this.events[eventName]) {
	      return new Error('事件无效');
	    }
        if (this.events[eventName]) {
            this.events[eventName] = this.events[eventName].filter(cb => cb != callback);
        }
    }
    // 只执行一次订阅的事件，然后移除
    once(eventName, callback) {
        // 绑定的时fn, 执行的时候会触发fn函数
        let fn = () => {
           callback(); // fn函数中调用原有的callback
           this.off(eventName, fn); // 删除fn, 再次执行的时候之后执行一次
        }
        this.on(eventName, fn);
    }
    // 触发事件的方法
    emit(eventName) {
        // 遍历执行所有订阅的事件
       this.events[eventName] && this.events[eventName].forEach(cb => cb());
    }
}


// 测试
let em = new EventEmitter();
let workday = 0;
em.on("work", function() {
    workday++;
    console.log("work everyday");
});

em.once("love", function() {
    console.log("just love you");
});

function makeMoney() {
    console.log("make one million money");
}
em.on("money",makeMoney);

let time = setInterval(() => {
    em.emit("work");
    em.off("money",makeMoney);
    em.emit("money");
    em.emit("love");
    if (workday === 5) {
        console.log("have a rest")
        clearInterval(time);
    }
}, 1000);
```
## 柯西化函数
柯里化（currying） 指的是将一个多参数的函数拆分成一系列函数，每个拆分后的函数都只接受一个参数。
```javascript
// 函数求和
function sumFn(...rest) {
    return rest.reduce((a, b) => a + b);
}
// 柯里化函数
var currying = function (func) {
    // 保存所有传递的参数
    const args = [];
    return function result(...rest) {
        // 最后一步没有传递参数，如下例子
        if(rest.length === 0) {
            return func(...args);
        } else {
            // 中间过程将参数push到args
            args.push(...rest);
            return result; // 链式调用
        }
    }
}

// 测试
currying(sumFn)(1)(2)(3)(4)(); // 10
currying(sumFn)(1, 2, 3)(4)(); // 10


// es6 实现
function curry(fn, ...args) {
  return fn.length <= args.length ? fn(...args) : curry.bind(null, fn, ...args);
}
```
## 实现instanceof
` instanceof `运算符用于检测构造函数的` prototype `属性是否出现在某个实例对象的原型链上。
```javascript
function isInstanceOf(instance, klass) {
  let proto = instance.__proto__;
  let prototype = klass.prototype;
  while (true) {
    if (proto === null) return false;
    if (proto === prototype) return true;
    proto = proto.__proto__;
  }
}

// 测试
class Parent {}
class Child extends Parent {}
const child = new Child()
console.log(isInstanceOf(child, Parent), isInstanceOf(child, Child), isInstanceOf(child, Array))
// true true false

```
## 实现promise.all
#### 1) 核心思路
接收一个 Promise 实例的数组或具有 Iterator 接口的对象作为参数
这个方法返回一个新的 promise 对象，
遍历传入的参数，用Promise.resolve()将参数"包一层"，使其变成一个promise对象
参数所有回调成功才是成功，返回值数组与参数顺序一致
参数数组其中一个失败，则触发失败状态，第一个触发失败的 Promise 错误信息作为 Promise.all 的错误信息。
#### 2）实现代码
一般来说，Promise.all 用来处理多个并发请求，也是为了页面数据构造的方便，将一个页面所用到的在不同接口的数据一起请求过来，不过，如果其中一个接口失败了，多个请求也就失败了，页面可能啥也出不来，这就看当前页面的耦合程度了
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if(!Array.isArray(promises)){
        throw new TypeError(`argument must be a array`)
    }
    var resolvedCounter = 0;
    var promiseNum = promises.length;
    var resolvedResult = [];
    for (let i = 0; i < promiseNum; i++) {
      Promise.resolve(promises[i]).then(value=>{
        resolvedCounter++;
        resolvedResult[i] = value;
        if (resolvedCounter == promiseNum) {
            return resolve(resolvedResult)
          }
      },error=>{
        return reject(error)
      })
    }
  })
}
// 测试
let p1 = new Promise(function (resolve, reject) {
    setTimeout(function () {
        resolve(1)
    }, 1000)
})
let p2 = new Promise(function (resolve, reject) {
    setTimeout(function () {
        resolve(2)
    }, 2000)
})
let p3 = new Promise(function (resolve, reject) {
    setTimeout(function () {
        resolve(3)
    }, 3000)
})
promiseAll([p3, p1, p2]).then(res => {
    console.log(res) // [3, 1, 2]
})
```
## 实现promise.race
该方法的参数是` Promise `实例数组, 然后其` then `注册的回调方法是数组中的某一个 Promise 的状态变为` fulfilled `的时候就执行. 因为 Promise 的状态只能改变一次, 那么我们只需要把` Promise.race `中产生的 Promise 对象的` resolve `方法, 注入到数组中的每一个 Promise 实例中的回调函数中即可.
```javascript
Promise.race = function (args) {
  return new Promise((resolve, reject) => {
    for (let i = 0, len = args.length; i < len; i++) {
      args[i].then(resolve, reject)
    }
  })
}
```

