# web-interview-questions
汇总前端面试题，包括不限于react、vue、JavaScript、HTML5、CSS等面试题


# ES6+
# JavaScript
# CSS
# 设计模式

# JavaScript手写代码题
## 实现map方法
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

// 方法2：
```
## 实现reduce方法
```javascript
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
// 防抖
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
```javascript

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
## 实现promise.race
```javascript

```
## 实现promise.all
```javascript

```
