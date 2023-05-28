---
title: 为什么我讨厌 JS 中的继承？
date: 2023-05-26 01:12:48
tags:
top:
---

# 使用继承

## 需求

### 现在要给 Person 添加功能

```js
person1.on('die', fn)
person1.emit('die')
person1.off('die', fn)

// 让 Person 实例具有发布订阅功能，怎么做？
```

### 简单，加代码

```js
class Person {
  constructor(name){
    this.name = name
  }
  seyHi(){
    console.log(`Hi, I'm ${this.name}`)
  }
  cache = []
  on(){}
  off(){}
  emit(){}
}
// 然后把以上代码写全
```

`let person1 = new Person()`，这样 `person1` 就既是人类，又能发布订阅

> 目前来说没有问题，因为代码没有重复

### 另一个类：报社

现在除了人类之外，报社也要拥有发布订阅功能

```js
class 报社{
  constructor(name){
    this.name = name
  }
  print(){}
  cache = []
  on(){}
  off(){}
  emit(){}
}
```

`let 报社1 = new 报社('新华日报')`，这样，`报社1` 就既是报社，又能发布订阅

> 那么问题来了，代码有重复了！

### 消除重复

#### Person 和报社有重复属性

*   把重复属性求出来，单独写个类 `EventEmitter`
*   然后让 `Person` 和 `报社` 继承 `EventEmitter`

#### 细节

*   `constructor` 要调用 `super()`
*   以保证 `EventEmitter` 实例被初始化

#### 代码

```js
class EventEmitter{
  constructor(){}
  cache = []
  on(){}
  off(){}
  emit(){}
}

class Person extends EventEmitter{
  constructor(name){
    super()
    this.name = name
  }
  seyHi(){
    console.log(`Hi, I'm ${this.name}`)
  }
}

class 报社 extends EventEmitter{
  constructor(name){
    super()
    this.name = name
  }
  print(){}
}
```

#### 继承的其他功能

重写

*   子类重写父类的所有属性，以实现多态
*   多态的意思是不同的子类对同一个消息有不同的反应

```js
class Person extends EventEmitter{
  constructor(name){
    super()
    this.name = name
  }
  seyHi(message){}
  on(eventName, fn){
    console.log('我要监听啦') // 人类的 on 会打印这句话，而报社不会，这就是多态
    super.on(eventName, fn)
  }
}
```

但是，继承有个问题。

# 继承有什么问题

如果我需要更多功能怎么办？

两个选择：

1.  让 `EventEmitter` 继承其他 `类`

    比如让 `发布订阅` 去继承 `跑`，那是不是很奇怪？

2.  让 `Person` 继承两个 `类`（多继承）

    很遗憾这个功能在 `JavaScript` 和 `Java` 中都没有，`C++` 中有

所以这两个方法都走不通

你可能会说: "那可以把这两个再抽象一下呀"，但是下面讲的的组合，你就会发现 `继承` 就是一个笑话！

# 使用组合

> `类` 是固定写法，把语法已经写死了，而 `组合` 没有固定写法，在这里我写我认为的组合写法，可能和你想的不一样，我这里只演示一种

## 让 Person 实现发布订阅

```js
class Person {
  eventEmitter = new EventEmitter() // 这句话的意思是：人类拥有发布订阅的功能
  name
  sayHi(){}
  on(eventName, fn){
    this.eventEmitter.on(eventName, fn)
  }
  off(eventName, fn){
    this.eventEmitter.off(eventName, fn)
  }
  emit(eventName, data){
    this.eventEmitter.emit(eventName, data)
  }
}

```

以上代码就是不使用继承，但拥有发布订阅的功能

> 这样功能是不是实现了？只不过目前的代码有点挫，但是可以优化呀

## 优化代码

```js
class Person{
  name
  seyHi(){}
}
let person1 = new Person('张三')
mixin(person1, new EventEmitter())

function mixin(to, from){
  for(let key in from){
    to[key] = from[key] // 把 from 上的所有属性拷贝到 to 上
  } 
}
// 注意，这是最简化的 mixin，实际会复杂一些
```

## 出现 Bug


![not_found_fn.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf6993e8f99a41ce9b2aa53d254903e8~tplv-k3u1fbpfcp-watermark.image?)

你会发现，`mixin` 之后，`EventEmitter` 里的函数却拷贝不过去，这是因为 `JS` 的 `class` 有一个毛病：他会把函数放在原型上面，就无法枚举

那么我们可以这么做：

改写 `EventEmitter`

```js
function createEventEmitter() {
  const cache = []
  return {
    on(){},
    off(){},
    emit(){}
  }
}

let eventEmitter = createEventEmitter()
let person1 = new Person()
mixin(person1, eventEmitter)
```

![使用组合.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d70d21babd14731ace26af9585dd441~tplv-k3u1fbpfcp-watermark.image?)

那就拥有了 `on()` `off()` `emit()`，当你把 `继承` 给干掉，你就会发现：这个世界扁平起来了

你需要什么，比如上面那 3 个 Api，我就把这 3 个 Api 的地址放到你身上，你想要其他功能，我就再把其他功能的地址放到你身上，这样就可以不断地去添加不同的功能

> 这就是 `组合` 的思想：你要什么东西，我就把什么东西的地址复制给你

## 如果你需要更多的功能

> 那就变得非常简单

```js
class Person {
  name
  sayHi(){}
}
let person1 = new Person('张三')
mixin(person1, new EventEmitter())
mixin(person1, new Flyer()) // person1 就可以飞了
mixin(person1, new Killer()) // person1 就可以开枪s人
```

你看！我还需要什么继承？而且组和这 3 个功能合集之间没有关系，它们的关系是扁平的

所以由此得出：有了组合，你可能不需要 class，直接使用：函数 + 闭包

## 变态需求

### 猫、狗、动物

```js
dog
  .wangwang() // 狗可以汪汪叫
  .poop() // 狗可以拉屎
cat
  .miaomiao() // 猫可以喵喵叫
  .poop() // 猫可以拉屎
```

那么猫和狗都可以拉屎，那我们就可以继承它们了

```js
// 伪代码，狗和猫除了都可以拉屎之外(继承了 poop())，还可以汪汪叫/喵喵叫
animal
  .poop()
    dog
      .wangwang()
    cat
      .miaomiao()
```

### 扫地机器人

> 那么动物会拉屎，那么这个时候需要一个机器人来打扫卫生

```js
cleaningRobot
  .run() // 可以跑
  .clean() // 可以打扫卫生

// 伪代码
animal
  .poop()
    dog
      .wangwang()
    cat
      .miaomiao()
```

### 两个机器人

> 我想让机器人可以s人

```js
murderRobot
  .run() // 可以跑
  .kill() // 可以s人
cleaningRobot
  .run()
  .clean()

// 伪代码
animal
  .poop()
    dog
      .wangwang()
    cat
      .miaomiao()
```

那么上面代码会发现，s人机器人和扫地机器人它们都可以跑，那么就可以抽象成以下代码：

```js
// 伪代码，s人机器人和扫地机器人都继承机器人可以跑的功能
robot
  .run()
    murderRobot
      .kill()
    cleaningRobot
      .clean()

// 伪代码
animal
  .poop()
    dog
      .wangwang()
    cat
      .miaomiao()
```

### 终极变态需求

> 那么我想要一个：狗形s人机器人

那么看看我们如何解决这个需求

```diff
# 伪代码
robot
# 跑我们需要
  .run()
    murderRobot
# s人我们需要
      .kill()
    cleaningRobot
      .clean()

// 伪代码
animal
# 拉屎我们不需要，机器人怎么能拉屎呢？
  .poop()
    dog
## 汪汪叫我们需要
      .wangwang()
    cat
      .miaomiao()
```

那么请问：这个需求我们如何用继承来解决？是不是很难？因为狗是继承了 animal 的拉屎呀！所以想用继承实现，是基本做不到的，或者说是非常难！

### 继承做不到，那么我们使用组合

那就太简单了，让我们把功能组合一下

*   `dog = poop() + wangwang()`
*   `cat = poop() + miaomiao()`
*   `cleaningRobot = run() + clean()`
*   `狗形s人机器人 = run() + kill() + wangwang()`

#### 不用 `class` 代码怎么写？

不用 `class` 写 `Dog`

```js
const createWang = state => ({
  wangwang: () => {
    console.log(`汪汪！我是${state.name}`)
  }
})
const createRun = state => ({
  run: () => state.position += 1
})
const createDog = name => {
  const state = {
    name,
    position: 0
  }
  return Object.assign(
    {},
    createWang(state),
    createRun(state)
  )
}
const dog = createDog('旺财')
```

那么现在运行一下以上代码：

![Dog.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/721131bd909d46e7abd8ccd9c3626551~tplv-k3u1fbpfcp-watermark.image?)
那我们来实现狗形s人机器人：

```js
const createMurderRobot = name => {
  const state = {
    name,
    position: 0
  }
  return Object.assign(
    {},
    createWang(state),
    createRun(state)
    createKill(state)
  )
}
const murderRobot = createMurderRobot('小黑')
```

你看！这种办法是不是让我们的代码组合非常的灵活？

这就是组合模式

# 运用场景

难道我们就不用继承了吗？

当然不是

## 什么时候用继承

### 场景

*   开发者不会用组合
*   功能没有太多交叉的地方，一眼就能看出继承关系
*   前端库比较喜欢用继承

### 举例

*   `const vm = new Vue({...})`
*   Vue 继承了 EventEmitter
*   `class App extends React.Component{...}`
*   React.Component 内置了方便的复用功能（钩子）

## 什么时候用组合

### 场景

*   功能组合非常灵活，无法一眼看出继承关系
*   插件系统
*   mixin 模式

#### 举例

*   `Vue.mixin()`
*   `Vue.use(MyPlugin)`
*   React 接受组件的组件

# 缺点

写法太灵活，我这里写的组合可能在其他人那里就不认同

# 总结

但不管怎么写，反正最终前端程序员基本上达成一个共识：组合是优于继承的

原因是组合能够实现扁平化的分配，而继承是垂直而，当出现交叉的时候，继承就基本做不到

# 一些吐槽

前端程序员，在很早之前是被 `Java` 程序员所占领的（没有人学前端，只有学 Java 的人最接近前端），Javaers 就把面向对象、class 带到前端

那个时候作为一名前端新人进到一个团队，那就看到面前有几位 Java 工程师转的前端，他们认为写代码应该用继承、用多态封装，然后你就抄他们的代码，然后抄着抄着发现面前的代码越写越复杂

因为前端它的特点就是 —— 需求很灵活，不像后端有些数据结构是非常固定的，前端你可能随时都能新增一些数据

这样的话前端就开始想：那会不会不是我的问题，而是面向对象的问题，为什么面向对象的继承这么难用，一旦出现交叉的数据，就搞不出来

所以就开始往其他的方向探索，其中非常大的方向就是函数式编程，函数式编程一般来说就不需要「经典的 class」

比如以上的代码，我就用函数来创造对象，我为什么要用 new？new 是一个前端程序员非常难以理解的东西

---

感谢阅读，下次见 :)
