---
title: 从零到实现 Redux 的过程
date: 2022-12-17 11:41:28
tags:
top:
---

[代码仓库](https://github.com/heycn/mini-redux)

个人愚见：`Redux` 的源码特别难看懂，也可能是 `Redux` 的源码是以实用为目的，看起来没什么头绪；

所以我一步一步实现一个 `Redux` 其中也有一些坑，最终效果和 `Redux` 的接口几乎是一致的，跟着以下思路，或许可以更容易理解 `Redux` 为什么要这么实现，包括每个概念

## 一、尝鲜使用 Vite4 初始化项目

> 今天是 Vite4 发布的第二天，我使用上了!\
> 2022-12-13

我已经准备了一个简单的 demo，它们的结构特别简单

- App 中有 3 个子元素，分别是：`Parent`、`Son`、`Grandson`
- Parent 负责 `展示 User 数据`，Son 负责 `修改 User 数据`

```jsx
import * as React from 'react'
const { useState, useContext } = React

const appContext = React.createContext(null)

export const App = () => {
  const [appState, setAppState] = useState({
    user: { name: 'heycn', age: 22 }
  })

  const contextValue = { appState, setAppState }

  return (
    <appContext.Provider value={contextValue}>
      <Parent />
      <Son />
      <Grandson />
    </appContext.Provider>
  )
}

// Parent 用于展示 User 数据
const Parent = () => (
  <section>
    Parent
    <User />
  </section>
)

// Son 用于修改 User 数据
const Son = () => (
  <section>
    Son
    <UserModifier />
  </section>
)

const Grandson = () => <section>Grandson Component</section>

const User = () => {
  const contextValue = useContext(appContext)
  return <div>UserName:{contextValue.appState.user.name}</div>
}

const UserModifier = () => {
  const { appState, setAppState } = useContext(appContext)
  const onChange = e => {
    appState.user.name = e.target.value
    setAppState({ ...appState })
  }
  return (
    <div>
      <input value={appState.user.name} onChange={onChange} />
    </div>
  )
}
```

## 二、使用 `useContext` 来读写数据

### 1. 数据从哪里来？

代码在上面 demo

- 使用 useState 声明了一个对象，里面包含 user
- 把 appState 和 setAppstate 封装成一个对象：contextValue
- 把 contextValue 放到 appContext.Provider 里
- appContext 是使用 React.createContext 创建的

### 2. 如何让 User 获取到 user 数据？

- 只需要两句话：
- 使用 useContext：`const contextValue = useContext(appContext)`
- 获取 `contextValue.appState.user.name`

### 3. 如何修改数据？

- 那么我们就需要调用 `setAppState`
- 看 `UserModifier` 里的 `onChange` 方法
- 注意：这个代码非常的不规范，后面会纠正它们！

## 三、`reducer` 的由来

> `reducer` 就是用来规范 state 创建流程的一个函数

[代码链接](https://github.com/heycn/mini-redux/commit/1cd2c87225712ef2b24c055dce1b4749cd101d05)

之前的代码，创建 `state` 的时候特别的不规范，它直接去修改了原始的 `state`

那么如何解决呢？\
提供一个函数来帮他去创建新的 `state`

### 1. `reducer` 雏形 —— `createNewState()`

> 目的：规范了创建流程

```jsx
// 接受3个参数：state(旧的state), actionType(操作)，actionData(新的state)
const createNewState = (state, actionType, actionData) => {
  if (actionType === 'update') {
    return {
      // 首先拷贝 user 之外的属性，然后创建一个 user
      ...state,
      user: {
        ...state.user, // 把之前user的其他属性拷贝过来
        ...actionData // 把给我额外对user的修改放在这里
      }
    }
  } else {
    return state
  }
}
```

### 2. 如何使用？

```diff
const UserModifiler = () => {
  const { appState, setAppState } = useContext(appContext)
  const onChange = e => {
-   appState.user.name = e.target.value
-   setAppState({ ...appState })
+  setAppState(createNewState(appState, 'updateUser', {name: e.target.value}))
}
```

### 3. reducer 接收两个参数！

> 那也简单

把 `actionType` 和 `actionData` 统一成一个叫 `action` 的东西，接受一个 `type` 和一个 `payload`

`payload` 其实就是 `data` 的意思

```diff
-const createNewState = (state, actionType, actionData) => {
+const createNewState = (state, {type, payload}) => {
- if (actionType === 'update') {
+ if (type === 'update') {
    return {
      ...state,
      user: {
        ...state.user,
-       ...actionData
+       ...payload
      }
    }
  } else {
    return state
  }
}
```

### 3. `reducer` 使用

```diff
const UserModifiler = () => {
  const { appState, setAppState } = useContext(appContext)
  const onChange = e => {
- setAppState(createNewState(appState, 'updateUser', {name: e.target.value}))
+ setAppState(createNewState(appState, {type: 'updateUser', payload: {name: e.target.value}}))
}
```

这样的话，`reducer` 就写出来了，是不是特别的简单呢？

这一点就只需记住一句话：`reducer` 是用来规范 `state` 创建流程的一个函数

## 四、dispatch

> 如何使用 `dispatch` 来规范 `setState` 的流程

[代码链接](https://github.com/heycn/mini-redux/commit/1d687e917800bc5a3d2b1c473b9f5c2308d67c11)

### 1. 太多重复的代码

首先来看一下我们之前是如何 `setState` 的:

`setAppState(reducer(appState, {type: 'updateUser', payload: {name: e.target.value}}))`

那如果我们要改 `user` 的 `age` 和 `height` 该怎么改？

```jsx
setAppState(reducer(appState, { type: 'updateAge', payload: { age: e.target.value } }))

setAppState(reducer(appState, { type: 'updateAge', payload: { age: e.target.value } }))
```

每次都要重复代码，那么下面将对其进行优化

### 2. 去除重复的代码

> dispatch 的由来

#### 第一步：实现 `dispatch`

我们写一个 `dispatch`

```jsx
const dispatch = action => {
  setAppState(reducer(appState, action))
}
```

使用:

```diff
const UserModifier = () => {
  const {appState, setAppState} = useContext(appContext)
  const onChange = e => {
-   setAppState(reducer(appState, {type: 'updateUser', payload: {name: e.target.value}}))
+   dispatch({type: 'updateUser', payload: {name: e.target.value}})
  }
}
```

你觉得这样子就可以了吗？\
并不能，因为 `UserModifier` 里的 `dispatch` 是没有办法访问到 `setAppState` 和 `appState` 的

React 规定：只能在组件内使用 `hooks`

至于出现这种情况，是由于我们把 `state` 放在了 `context` 里，如果 `state` 不在 `context` 里，那就好办了，但是那种改动太大了

那我们想想，就以现在的办法如何实现：让 `dispatch` 访问到 `state` 和 `setState`

#### 第二步：实现让 `dispatch` 访问到 `state` 和 `setState`

思路：我们用一个组件来包住 `dispatch`，然后把 `dispatch` 再给需要使用的组件

```jsx
// 使用 Wrapper 来包住 UserModifier
// 注意之前使用到 <UserModifier /> 要改为 <Wrapper />

const Wrapper = () => {
  return <UserModifier />
}
```

```jsx
const Wrapper = () => {
  const { appState, setAppState } = useContext(appContext) // 使 dispatch 可以使用上下文
  // 把 dispatch 放到 Wrapper 里
  const dispatch = action => {
    setAppState(reducer(appState, action))
  }
  // 把 dispatch 和 state 传给 UserModifier
  return <UserModifier dispatch={dispatch} state={appState} />
}

const UserModifier = ({ dispatch, state }) => {
  const onChange = e => {
    dispatch({
      type: 'updateUser',
      payload: { name: e.target.value }
    })
  }
  return (
    <div>
      <input value={state.user.name} onChange={onChange} />
    </div>
  )
}
```

想要读数据，就从 `props` 里面读 `state`，想要写数据就从 `props` 里使用 `dispatch`

目前，我们就完成了 `UserModifier` 一个组件的封装，它可以通过 `props` 来读写全局数据

所有人，直接调 `dispatch`，不要用去多写那三个单词了

实际上这个功能不是由 `redux` 实现的，是由 `react-redux` 实现的，但是大家用的时候都是一起用的，这里就不做区分了

## 五、高阶组件 `connect`

> 让组件与全局状态连接起来\
> 原理：函数里接收一个组件，返回一个新的组件

[代码链接](https://github.com/heycn/mini-redux/commit/cab085bb9651181152e2789eaf0e5be0821da2b9)

在上面代码，我们是把 `UserModifier` 包装成了 `Wrapper`，用的时候我们是一定要使用 `Wrapper`，因为如果直接使用 `UserModifier` 是得不到 `dispatch` 和 `state`，因此，我们任何一个组件想要读取全局 `state`，都需要封装成一个 `Wrapper`，那如果有 100 个组件，难道都要重新写 100 遍吗？当然不是这样子的

所以我们需要声明一个函数来实现，用来自动创建之前的 `Wrapper`

```jsx
// connect
const connect = Component => {
  return props => {
    const { appState, setAppState } = useContext(appContext)
    const dispatch = action => {
      setAppState(reducer(appState, action))
    }
    return <Component {...props} dispatch={dispatch} state={appState} />
  }
}

// 使用
const UserModifier = connect(({ dispatch, state, children }) => {
  const onChange = e => {
    dispatch({
      type: 'updateUser',
      payload: { name: e.target.value }
    })
  }
  return (
    <div>
      {children}
      <input value={state.user.name} onChange={onChange} />
    </div>
  )
})
```

如果你看 `redux` 提供的 `connect`，你会发现它接收的参数比我上面的组件还多，后面我们接着实现！

## 六、避免多余的 render

[代码链接](https://github.com/heycn/mini-redux/commit/83b79bbdc7cb67d310eda5c9cccd020c5dcba209)

我们在之前的代码里每个组件都加上 log

```jsx
const Brother = () => {
  console.log('Brother render!')
  return (
    <section>
      Brother
      <User />
    </section>
  )
}

const Sister = () => {
  console.log('Sister render!')
  return (
    <section>
      Sister
      <UserModifier />
    </section>
  )
}

const Cousin = () => {
  console.log('Cousin render!')
  return <section>Cousin</section>
}

const User = () => {
  const contextValue = useContext(appContext)
  console.log('User render!')
  return <div>UserName:{contextValue.appState.user.name}</div>
}

const UserModifier = connect(({ dispatch, state, children }) => {
  const onChange = e => {
    dispatch({
      type: 'updateUser',
      payload: { name: e.target.value }
    })
  }
  console.log('UserModifier render!')
  return (
    <div>
      {children}
      <input value={state.user.name} onChange={onChange} />
    </div>
  )
})
```

我们会发现：我们只改一个组件，而上面 5 个组件都会重新 render，这样子我们只要改动 `state` 中的一小点，就会导致整个应用的重新执行

我们希望用到的时候，才 render

我们来看下问题是如何产生的：

- 当我们改变 `input` 的值的时候，它会调用 `setAppState`
- 是通过 `dispatch` 调用到的
- `dispatch` 是由 `context` 来的
- 而 `context` 最初是从 `AppContext` 拿到的
- 根据 React 规定：只要调用到这个组件的 `setState`，并且给 `setState`，传的是一个新对象，那么这个组件就一定会重新渲染

首先我们想到的是使用 `useMemo`，这样能避免组件重新执行，但是，这么写太麻烦了，那么 `redux` 会设计一种机制：只有用到 `state` 里某个属性的地方，在这个属性变化的时候，再重新执行

实现思路及过程：

1. 首先我们把 `setState` 移除掉，因为它必然会导致组件执行
2. 我们创建一个对象 `store`，里面有 `state` 和 `setState`
   ```jsx
   const store = {
     state: {
       // 用于存放数据
       user: { name: 'heycn', age: 22 }
     },
     setState(newState) {
       // 用于修改数据
       store.state = newState
     }
   }
   ```
3. 把之前用到 `useState` 的地方都改为 `store` 的 `state` 和 `setState`
4. 目前展示没问题，但是修改无法显示，其实数据已经在 `store` 改变了，只是我们没有调用 `react` 的 `useState`，那么我们让他强制刷新
5. 在 `connect` 里使用 `const [, update] = useState({})`，它的值为一个空对象，然后再 `dispatch` 里调用 `update({})`，这样的话，被 `connect` 的组件就会强制刷新
6. 但是这样的话，其他使用到的 `state` 的组件，就无法更新，所以我们需要去订阅一下变化
7. 在 `store` 里创建 `subscribe` 函数，我们可以让每个组件订阅 `state` 的变化
   ```jsx
   const store = {
     state: {
       user: { name: 'heycn', age: 22 }
     },
     setState(newState) {
       store.state = newState
       // 每次 setState 就告诉订阅者
       store.listeners.map(fn => fn(store.state))
     },
     listeners: [], // 把所有订阅的监听者放进来
     subscribe(fn) {
       store.listeners.push(fn)
       return () => {
         // 取消订阅
         const index = store.listeners.indexOf(fn)
         store.listeners.splice(index, 1)
       }
     }
   }
   ```
8. 在 `connect` 中使用 `useEffect`，只在组件第一次渲染时订阅，调用 `useState` 的 `update()`

以下是完整代码

```jsx
import * as React from 'react'
const { useState, useEffect, useContext } = React

const appContext = React.createContext(null)

const connect = Component => {
  return props => {
    const { state, setState } = useContext(appContext)
    const [_, forceUpdate] = useState({})
    useEffect(() => {
      store.subscribe(() => {
        forceUpdate({})
      })
    }, [])
    const dispatch = action => {
      setState(reducer(state, action))
    }
    return <Component {...props} dispatch={dispatch} state={state} />
  }
}

const store = {
  state: {
    user: { name: 'heycn', age: 22 }
  },
  setState(newState) {
    store.state = newState
    store.listeners.map(fn => fn(store.state))
  },
  listeners: [],
  subscribe(fn) {
    store.listeners.push(fn)
    return () => {
      const index = store.listeners.indexOf(fn)
      store.listeners.splice(index, 1)
    }
  }
}

export const App = () => {
  return (
    <appContext.Provider value={store}>
      <Brother />
      <Sister />
      <Cousin />
    </appContext.Provider>
  )
}

// Brother 用于展示 User 数据
const Brother = () => {
  console.log('Brother render!')
  return (
    <section>
      Brother
      <User />
    </section>
  )
}

// Sister 用于修改 User 数据
const Sister = () => {
  console.log('Sister render!')
  return (
    <section>
      Sister
      <UserModifier />
    </section>
  )
}

const Cousin = () => {
  console.log('Cousin render!')
  return <section>Cousin</section>
}

const User = connect(({ state, dispatch }) => {
  console.log('User render!')
  return <div>UserName:{state.user.name}</div>
})

const reducer = (state, { type, payload }) => {
  if (type === 'updateUser') {
    return {
      ...state,
      user: {
        ...state.user,
        ...payload
      }
    }
  } else {
    return state
  }
}

const UserModifier = connect(({ dispatch, state }) => {
  const onChange = e => {
    dispatch({
      type: 'updateUser',
      payload: { name: e.target.value }
    })
  }
  console.log('UserModifier render!')
  return (
    <div>
      <input value={state.user.name} onChange={onChange} />
    </div>
  )
})
```

## 七、Redux 雏形

将代码抽离，把 `redux` 有关的代码放在同一个文件

[代码链接](https://github.com/heycn/mini-redux/commit/d55efbb082f5944bb51f76a7dbdf1d3b815dc83a)

## 八、让 connect 支持 selector

> `react-redux` 提供的 `selector`

[代码链接](https://github.com/heycn/mini-redux/commit/1abb94588a168de71addec5446fa8a421c4df75d)

这是一个选择函数，比如：

```jsx
const User = connect(state => {
  return { user: state.user }
})(({ user }) => {
  return <div>Use: {user.name}</div>
})
```

以下是 api 的实现步骤：

1. 来到 `redux.js` 里 的 `connect`
2. 我们给他添加一个参数，表示先接受一个参数，再接受第二个参数
   ```diff
   - const connect = Component => {
   + const connect = selector => Component => {
     return props => {
       const { state, setState } = useContext(appContext)
       const [_, forceUpdate] = useState({})
   +   const data = selector ? selector(state) : {state}
       useEffect(() => {
         store.subscribe(() => {
           forceUpdate({})
         })
       }, [])
       const dispatch = action => {
         setState(reducer(state, action))
       }
   -   return <Component {...props} dispatch={dispatch} state={state} />
   +   return <Component {...props} {...data} dispatch={dispatch} />
     }
   }
   ```

我们通过一些简单的代码就实现了 `selector`，他还有其他非常重要的作用，请看下面！

## 九、实现精准渲染

> 使用 `selector` 来实现 `精准渲染`\
> 组件只在自己的数据变化时 `render`

[代码链接](https://github.com/heycn/mini-redux/commit/b5468b8b9d5409915b11e9ab905cafe7ed77209a)

### 问题：

1. 我们在 `store` 里添加：

   ```diff
   state: {
     user: { name: 'heycn', age: 22 },
   + educational: { school: 'Tsinghua University' }
   }
   ```

2. 然后在 `Cousin` 读取新添加的数据
   ```diff
   -const Cousin = () => {
   +const Cousin = connect(state => {
   + return { educational: state.educational }
   +})(() => {
     return (
       <section>
         <h1>Cousin</h1>
   +     <div>educational: {educational.school}</div>
       </section>
     )
   })
   ```
3. 然后让我们修改 `user` 时，`Cousin` 也会重新渲染，而 `Cousin` 里只使用到 `educational`

### 如何解决

1. 那我可以在 `Cousin` 做一个检查：如果 `Cousin` 没有更新，我们就不去重新渲染 `Cousin`
2. 但是这是存在一个逻辑悖论的，因为：如果 `Cousin` 要去做检查，那么这个时候 `Cousin` 就已经执行了，我们可以在 `connect` 里做手脚
3. 在 `connect` 里我们返回一个组件，我们叫它为 `Wrapper`，这个 `Wrapper` 的作用很大，我们可以在 `Wrapper` 里面做检查：
4. 如果被选择的 `selectedState` 没有改变，我们就不去做渲染

   ```diff
   +const changed = (oldState, newState) => {
   +  let changed = false
   +  for (let key in oldState) {
   +    if (oldState[key] !== newState[key]) {
   +      changed = true
   +      break
   +    }
   +  }
   +  return changed
   +}

   export const connect = selector => Component => {
     return props => {
       const { state, setState } = useContext(appContext)
       const [_, forceUpdate] = useState({})
       const selectedState = selector ? selector(state) : { state }
       useEffect(() => {
         store.subscribe(() => {
   +       const selectedState = selector ? selector(state) : { state }
   +       if (changed(selectedState, newSelectedState)) {
             forceUpdate({})
   +       }
         })
   -   }, [])
   +   }, [selector])
       const dispatch = action => {
         setState(reducer(state, action))
       }
       return <Component {...props} {...selectedState} dispatch={dispatch} />
     }
   }
   ```

5. 但是，我们需要取消订阅，不然可能在意想不到的时候不停地订阅，所以需要进行取消订阅，由于订阅里面返回了取消订阅，所以只需要这么做：
   ```diff
   export const connect = selector => Component => {
     return props => {
       const { state, setState } = useContext(appContext)
       const [_, forceUpdate] = useState({})
       const selectedState = selector ? selector(state) : { state }
       useEffect(() => {
   +     return {
           store.subscribe(() => {
             const selectedState = selector ? selector(state) : { state }
             if (changed(selectedState, newSelectedState)) {
               forceUpdate({})
             }
           })
   +     }
       }, [selector])
       const dispatch = action => {
         setState(reducer(state, action))
       }
       return <Component {...props} {...selectedState} dispatch={dispatch} />
     }
   }
   ```

这样就实现精准渲染：组件只在自己的数据变化时 `render`！

## 十、`connect` 的第二个参数：`mapDispatchToProps`

### api 设计

我们期望 api 是这么使用的：

```jsx
const UserModifier = connect(null, dispatch => {
  return {
    updateUser: attrs => dispatch({ type: 'updateUser', payload: attrs })
  }
})(({ updateUser, state }) => {
  const onChange = e => {
    updateUser({ name: e.target.value })
  }
  console.log('UserModifier render!')
  return (
    <div>
      <input value={state.user.name} onChange={onChange} />
    </div>
  )
})
```

### 代码实现

```diff
export const connect = selector => Component => {
  return props => {
    const { state, setState } = useContext(appContext)
    const [_, forceUpdate] = useState({})
+   const dispatch = action => {
+     setState(reducer(state, action))
+   }
    const selectedState = selector ? selector(state) : { state }
+   const selectedDispatches = mapDispatchToProps ? mapDispatchToProps(dispatch) : { dispatch }
    useEffect(() => (
      store.subscribe(() => {
        const newSelectedState = selector ? selector(store.state) : { state: store.state }
        if (changed(selectedState, newSelectedState)) {
          forceUpdate({})
        }
      })
    ), [selector])
-   const dispatch = action => {
-     setState(reducer(state, action))
-   }
-   return <Component {...props} {...selectedState} dispatch={dispatch} />
+   return <Component {...props} {...selectedState} {...selectedDispatches} />
  }
}
```

## 十一、connect 的意义

我们会发现 `connect` 函数的调用形式很奇怪，我们来看看究竟是在考虑什么！

看这里代码的 diff 就明白了：[代码链接](https://github.com/heycn/mini-redux/commit/93d4fb2a9dc59bdd968759548f1cc5bb2a8e7516)

`mapStateToProps` 是用来封装写，`mapDispatchToProps` 是用来封装读，所以 `connect` 是用来封装 `读` 和 `写`，也就是封装一个资源，你可以对这个资源进行读写，然后只要再传一个组件就行了，之所以要分成两次调用，就是为了方便：你先调用一次得到一个 “半成品”，这个 “半成品” 可以跟任何组件相结合，它会把 `读`、`写` 接口传给任何组件，然后等你想用一个组件的时候，就可以调不同的组件

这就是 `connect` 的意义

## 十二、封装 Provider 和 createStore

### createStore

它接受两个参数，一个 `reducer` 一个 `initState`

看代码 `diff` 即可知道如何封装：[代码链接](https://github.com/heycn/mini-redux/commit/583268fc277bf730b5ecc3544dffc3c73d07f451)

### 封装 `Provider`

redux 官方的使用方式是这样子的：`<Provider store={store}></Provider>`，那我们只需要分装成一个组件即可：

```jsx
export const Provider = ({ store, children }) => {
  return <appContext.Provider value={store}>{children}</appContext.Provider>
}
```

目前为止，目前我们的封装的 `redux` 和 官方的 `redux` 的接口，几乎是一致的，可能会有一些细微的区别，通过手写 `redux`，我们基本可以理解 `redux` 的实现，让我们总结一下

## 十三、Redux 概念总结（精髓）

让我们来了解，`redux` 和 `react-redux` 的主要思路

请配合代码阅读

### 主要思路

- 首先我们有一个 `App` 组件，里面包含很多组件，我们需要让每一个组件都可以访问到一个全局的 `state`
- `state` 是我们的第一个概念：`state` 放在哪里呢？`redux` 是把他放在 `store` 里的
- 让组件和 `store` 的 `state` 连接起来：`react-redux` 提供的一个 api —— `connect`，用于连接组件和 `state`
- `store` 的 `state` 连接之后做什么：连接之后就是 `读` 和 `写`
- `读` 操作：从组件的属性里面取 `state`，如果想读得更精确，可以传一个 `mapStateToProps`
- `写` 操作：从组件的属性里面取 `dispatch`，如果想写得更精确，可以传一个 `mapDispatchToProps`，可以用来封装 `api`，你可以对这个 `api` 的资源进行读写，然后只要再传一个组件就行了

### 回顾 `connect` 作用

- `connect` 实际上是对组件进行一封装，我称这个组件为 `Wrapper`，然后把 `Wrapper` 返回出去，主要做了 3 件事情
- 第一件事 `获取读写接口`：从上下文拿到 `state` 和 `setState`（是 `store` 的），也就是拿到 `读` 和 `写` 接口，实际上不用上下文也行，直接从 `store` 也行
- 第二件事 `封装读写接口`：进行封装，比如根据 `mapStateToProps` 得到具体的数据和具体的 `mapDispatchToProps`
- 第三件事 `订阅store更新`：在恰当的时候进行更新，对 `store` 进行订阅，只要 `store` 变化，就会在数据变更的情况下调用 `forceUpdate` 进行强制更新组件，这里的 `forceUpdate` 是我做的一个小技巧
- 最后就返回 `Wrapper` 这个组件

简单来讲，就是：

1. 获取读写接口
2. 封装读写接口
3. 订阅 `store` 更新，如果 `store` 更新了，就更新组件
4. 返回这个组件

### 总结

现在我们已经知道 `store`、`state`、`dispatch`、`connect`、`Provider` 的概念了

`dispatch` 这里又可以分出几个概念，比如：

- `reducer`：这个很难具体的解释，但我把他认为是规范创建 `state` 的过程，因为每次更新 `state` 我们不能改原来的 `state`，要创建新的 `state`，所以他是创建 `state` 的过程
- `initState`：这个好理解，就是初始的 `state`
- `action`：变动的描述。因为 `reducer` 是接受一个 `state`，一个 `action` 然后返回一个新的 `state`；所以是这一次变动的描述；比如 `action` 的类型、`payload` 可以是 `store` 的具体的各种信息

到这里，redux 的大部分概念我们都彻底的了解了

## 十四、API 封装技巧

看代码 `diff`

[代码链接：state -> getState](https://github.com/heycn/mini-redux/commit/7a91991f2d9e9d96ce9adc2d4050defcb69a7752)

[代码链接：隐藏 reducer](https://github.com/heycn/mini-redux/commit/16b82f7cb991dd8d30a8edb0fb1b4c0f60f88aeb)

[代码链接：隐藏 listeners](https://github.com/heycn/mini-redux/commit/781a9c78224f09fa9ce8adcf32bdeb0e97f98cc4)

[代码链接：隐藏 setState，封装 dispatch](https://github.com/heycn/mini-redux/commit/becbe9eaa893da713a18fe8ba7845918394f410c)

## 十五、让 Redux 支持函数 Action

目前我们的 redux 是不支持异步 `Action` 的，我们来看下如果要做异步的 `Action` 的话，我们应该怎么做，只做了以下几步

[代码链接](https://github.com/heycn/mini-redux/commit/795a158b3347e19c37bc7648df7a8a3aaa99aa4e)

```jsx
let { dispatch } = store

const preDispatch = dispatch

dispatch = action => {
  action instanceof Function ? action(dispatch) : preDispatch(action)
}
```

## 十六、让 Redux 支持 PromiseAction

### api 设计

```jsx
dispatch({ type: 'updateUser', payload: ajax('/user').then(response => response.data) })
```

### 代码实现

[代码链接](https://github.com/heycn/mini-redux/commit/83af80aca9856f4cb4ac606fb2c9b397ec5bd4a9)

```jsx
const asyncDispatch = dispatch

dispatch = action => {
  action.payload instanceof Promise
    ? action.payload.then(data => {
        dispatch({ ...action, payload: data })
      })
    : asyncDispatch(action)
}
```

## 十七、中间件 `redux-thunk` 和 `redux-promise` 原理

一个 `Middleware` 就是一个函数，这个函数可以去修改 `dispatch`

这两个中间件可以让 `redux` 支持异步 `action`

### `redux-thunk`

如果 `action` 是一个函数，就调用它，否则就进入下一个函数

### `redux-promise`

如果 `payload` 是一个函数，就在 `Promise` 后面接上一个 `then` 和 `catch`，否则就进入下一个函数

---

感谢阅读，下次见 :)
