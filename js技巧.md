###### 1、深拷贝

``` javascript
const mapTag = '[object Map]';
const setTag = '[object Set]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';
const argsTag = '[object Arguments]';

const boolTag = '[object Boolean]';
const dateTag = '[object Date]';
const numberTag = '[object Number]';
const stringTag = '[object String]';
const symbolTag = '[object Symbol]';
const errorTag = '[object Error]';
const regexpTag = '[object RegExp]';
const funcTag = '[object Function]';

const deepTag = [mapTag, setTag, arrayTag, objectTag, argsTag];


function forEach(array, iteratee) {
  let index = -1;
  const length = array.length;
  while (++index < length) {
    iteratee(array[index], index);
  }
  return array;
}

function isObject(target) {
  const type = typeof target;
  return target !== null && (type === 'object' || type === 'function');
}

function getType(target) {
  return Object.prototype.toString.call(target);
}

function getInit(target) {
  const Ctor = target.constructor;
  return new Ctor();
}

function cloneSymbol(targe) {
  return Object(Symbol.prototype.valueOf.call(targe));
}

function cloneReg(targe) {
  const reFlags = /\w*$/;
  const result = new targe.constructor(targe.source, reFlags.exec(targe));
  result.lastIndex = targe.lastIndex;
  return result;
}

function cloneFunction(func) {
  const bodyReg = /(?<={)(.|\n)+(?=})/m;
  const paramReg = /(?<=\().+(?=\)\s+{)/;
  const funcString = func.toString();
  if (func.prototype) {
    const param = paramReg.exec(funcString);
    const body = bodyReg.exec(funcString);
    if (body) {
      if (param) {
        const paramArr = param[0].split(',');
        return new Function(...paramArr, body[0]);
      } else {
        return new Function(body[0]);
      }
    } else {
      return null;
    }
  } else {
    return eval(funcString);
  }
}

function cloneOtherType(targe, type) {
  const Ctor = targe.constructor;
  switch (type) {
    case boolTag:
    case numberTag:
    case stringTag:
    case errorTag:
    case dateTag:
      return new Ctor(targe);
    case regexpTag:
      return cloneReg(targe);
    case symbolTag:
      return cloneSymbol(targe);
    case funcTag:
      return cloneFunction(targe);
    default:
      return null;
  }
}

function clone(target, map = new WeakMap()) {

  // 克隆原始类型
  if (!isObject(target)) {
    return target;
  }

  // 初始化
  const type = getType(target);
  let cloneTarget;
  if (deepTag.includes(type)) {
    cloneTarget = getInit(target, type);
  } else {
    return cloneOtherType(target, type);
  }

  // 防止循环引用
  if (map.get(target)) {
    return map.get(target);
  }
  map.set(target, cloneTarget);

  // 克隆set
  if (type === setTag) {
    target.forEach(value => {
      cloneTarget.add(clone(value, map));
    });
    return cloneTarget;
  }

  // 克隆map
  if (type === mapTag) {
    target.forEach((value, key) => {
      cloneTarget.set(key, clone(value, map));
    });
    return cloneTarget;
  }

  // 克隆对象和数组
  const keys = type === arrayTag ? undefined : Object.keys(target);
  forEach(keys || target, (value, key) => {
    if (keys) {
      key = value;
    }
    cloneTarget[key] = clone(target[key], map);
  });

  return cloneTarget;
}

console.log(clone({
  name: '张三', age: 23,
  obj: { name: '李四', age: 46 },
  arr: [1, 2, 3]
})) // { name: '张三', age: 23, obj: { name: '李四', age: 46 }, arr: [ 1, 2, 3 ] }
```
###### 2.拦截对象
利用Object.defineProperty拦截对象
无法拦截数组的值
``` javascript
let obj = { name: '', age: '', sex: '' },
  defaultName = ["这是姓名默认值1", "这是年龄默认值1", "这是性别默认值1"];
Object.keys(obj).forEach(key => {
  Object.defineProperty(obj, key, { // 拦截整个object 对象，并通过get获取值，set设置值，vue 2.x的核心就是这个来监听
    get() {
      return defaultName;
    },
    set(value) {
      defaultName = value;
    }
  });
});

console.log(obj.name); // [ '这是姓名默认值1', '这是年龄默认值1', '这是性别默认值1' ]
console.log(obj.age); // [ '这是姓名默认值1', '这是年龄默认值1', '这是性别默认值1' ]
console.log(obj.sex); // [ '这是姓名默认值1', '这是年龄默认值1', '这是性别默认值1' ]
obj.name = "这是改变值1";
console.log(obj.name); // 这是改变值1
console.log(obj.age);  // 这是改变值1
console.log(obj.sex); // 这是改变值1

let objOne = {}, defaultNameOne = "这是默认值2";
Object.defineProperty(obj, 'name', {
  get() {
    return defaultNameOne;
  },
  set(value) {
    defaultNameOne = value;
  }
});
console.log(objOne.name); // undefined
objOne.name = "这是改变值2";
console.log(objOne.name); // 这是改变值2

```
利用proxy拦截对象

``` javascript
let obj = { name: '', age: '', sex: '' }
let handler = {
  get(target, key, receiver) {
    console.log("get", key); 
    return Reflect.get(target, key, receiver);
  },
  set(target, key, value, receiver) {
    console.log("set", key, value); // set name 李四  // set age 24
    return Reflect.set(target, key, value, receiver);
  }
};
let proxy = new Proxy(obj, handler);
proxy.name = "李四";
proxy.age = 24;
```
defineProterty和proxy的对比：
1.defineProterty是es5的标准,proxy是es6的标准;
2.proxy可以监听到数组索引赋值,改变数组长度的变化;
3.proxy是监听对象,不用深层遍历,defineProterty是监听属性;
4.利用defineProterty实现双向数据绑定(vue2.x采用的核心)


