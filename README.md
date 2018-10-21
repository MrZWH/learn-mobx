## 学习 mobx

### 核心思想
- 状态变化引起的副作用应该被自动触发
- action -> state -> reaction

### decorator
支持需安装：
```
npm install babel-plugin-transform-decorators-legacy -D
```
```js
function log(target) {
  const desc = Object.getOwnPropertyDescriptors(target.prototype)
  for(const key of Object.keys(desc)) {
    if (key === 'constructor') {
      continue;
    }

    const func = desc[key] .value;

    if ('function' === typeof func){
      Object.defineProperty(target.prototype, key, {
        value(...args) {
          console.log('before' + key)
          const ret = func.apply(this, args)
          console.log('after' + key)
          return ret
        }
      })
    } 
  }
}

// 类属性修饰器
function readonly(target, key, descriptor) {
  descriptor.writable = false;
}

// 类方法修饰器
function validate(target, key, descriptor) {
  const func = descriptor.value;
  descriptor.value = function(...args) {
    for (let num of args) {
      if ('number' !== typeof num) {
        throw new Error(`"${num} is not a number"`)
      }
    }

    return func.apply(this, args)
  }
}

@log
class Numberic {
  @readonly PI = 3.1415926;

  @validate
  add(...nums) {
    return nums.reduce((p, n) => (p + n), 0)
  }
}

new Numberic().add(1, 2)
```

### mobx 常用 API 可观察的数据（observable）

#### 什么是 observable
- 是一种让数据的变化可以被观察的方法

```js
import {observable, isArrayLike} from 'mobx'

// array object map
const arr = observable(['a', 'b', 'c'])

console.log(arr, Array.isArray(arr), isArrayLike(arr), arr[0], arr[1])

// observable.box 包装基本数据类型
var num = observable.box(20)
var str = observable.box('hello')
var bool = observable.box(true)

num.set(50)

console.log(num.get())
```
