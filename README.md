## 走近 mobx

### 核心思想
- 状态变化引起的副作用应该被自动触发
- action -> state -> reaction

### 巩固 class 类语法
需安装：
```
npm i webpack webpack-cli babel-core babel-preset-env babel-loader -D
```
定义类的成员属性时，要识别语法需安装：
```
npm i babel-plugin-transform-class-properties -D
```

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


class Store {
  @observable array = []
  @observable obj = {}
  @observable map = new Map()
  
  @observable string = 'hello'
  @observable number = 29
  @observable bool = true
  
}
```

### 对可观察的数据作出反应
观察数据的变化方式
- computed
- autorun
- when
- Reaction

```js
import {observable, computed, autorun, when, reaction} from 'mobx'

class Store {
  @observable array = []
  @observable obj = {}
  @observable map = new Map()
  
  @observable string = 'hello'
  @observable number = 29
  @observable bool = true
  
  @computed get mixed() {
    return store.string + '/' + store.number
  }
}

//computed
var store = new Store();

// var foo = computed(function() {return store.string + '/' + store.number})

// foo.observe(function(change){
//   console.log(change)
// })

// store.string = 'world';
// store.number = 39;

// autorun
// autorun(() => {
//   console.log(store.string + '/' + store.number)
// })

// when
// when(() => store.bool, () => console.log("it's true"))

// store.bool = true

// reaction
reaction(() => [store.string, store.number], arr => console.log(arr.join('/')))
// store.string = 'world';
// store.number = 39;
```

### 修改可观察数据(action)

```js
import {runInAction, observable, computed, autorun, when, reaction, action} from 'mobx'

class Store {
  @observable array = []
  @observable obj = {}
  @observable map = new Map()
  
  @observable string = 'hello'
  @observable number = 29
  @observable bool = true
  
  @computed get mixed() {
    return store.string + '/' + store.number
  }
  
  @action bar() {
    this.action = 'world';
    this.number = 30
  }
}

var store = new Store()

reaction(() => [store.string, store.number], arr => console.log(arr.join('/')))

// var bar = store.bar
// bar()

runInAction('modify', () => {
  store.action = 'world';
  store.number = 30
})
```

### mobx 应用使用 mobx-react
```
npm i babel-preset-react -D
```

## 实现一个 TODOlist

## 最佳实践

### 常用的工具函数
- observe
- toJS
- trace
- spy

### 提升性能
三法则：
- 细粒度拆分视图组件
- 使用专用组件处理列表
- 尽可能晚的解构可观察数据
