---
title: 我的请求封装思路 - HttpClient
date: 2022-6-15 22:27:42
tags:
top:
---

> 这是我的封装思路，请根据口味酌情选择

前端 hppt 请求目前比较流行 `Axios`，但实际上不管使用什么请求库，封装思路都是一样的

我不确定我以后的请求库会不会改变，而且脑子里有个思路(具体在下面)，所以我在这里封装了一个中间层 ——— HttpClient

目的是可以隔离对 Axios 的依赖

# 思路

> 封装目的就是 DRY：Dont Repeat Yourself！

除了具体的业务，其他的逻辑都应该放在封装层\
输出一个对象，而这个对象符合当前业务的所有需求

## 一. 封装 HttpClient 中间层

> 虽然说，现在请求库是用的 Axios，但是，我做一个中间层，不直接用 Axios，我使用一个叫 HttpClient 的类，对 Axios 进行一次封装

这样的好处有两个：

1. 增强对代码的控制：HttpClient 中间层是由我封装的，会比较符合我的习惯，增强对代码的控制
2. 可以修改参数：Axios 的参数怎么传都写死了，如果我们有个中间层来做一些适配，就算有一天我不用 Axios 了，比如：fetch、useRequest、SWR... 我们的代码都不用改，我们只需要改 HttpClient 这个类

当然，我只是建议

## 二. 统一的错误处理

那我们有哪些错误需要处理呢？

比如我们发送请求太频繁了，会得到一个 429 状态码，那我们只要看到这个状态码，我们就直接提示 “你的发送得频繁啦！”

再比如，401,，这是用户请求了一个接口，但是未登录，有可能就是那一瞬间，登录过期了，那我们一收到 401 状态码，我们就可以提示用户，问用户登录过期或者未登录，要不要登录，确定或取消

再比如，403，这是没有权限，那这一样也是弹个框，或者跳转某个页面

再比如，500 502 503，我们可以直接提示：“抱歉，服务器繁忙，请稍后再试”

当然，还有其他很多状态码，这不重要，重要的是我们有个地方可以统一地进行处理

## 三. 加载中

我们很多时候希望用户点击提交，或点击发送的时候，出现一个 loading 的图案，可以让用户耐心的等待，免得烦躁

那我们的请求开始，总会有请求结束的时候，我们何妨不统一做一个处理呢？

## 四. 其他

其他的还有：权限控制、Token... 等，思路都和上面一样，这里主要就看个人的项目需求了

# 实现

## 一. 创建 HttpClient

### 1. 找一个文件夹，创建一个叫 `HttpClient.ts` 的文件，当然，名字的定义无所谓

### 2. 然后在文件中写：

```ts
export class Http {}
```

> 为什么我使用 class？\
> 这个无所谓了，用函数也行，我用 class 是因为里面还可以再封装一个函数，酌情选择吧

### 3. 统一接口

> 我们业务大部分都是增删改查，那我就只针对这个写了

```ts
export class Http {
  get() {}
  post() {}
  patch() {}
  delete() {}
}
```

- 为什么不写 put？\
  其实 `put`、`patch`、`post` 他们非常类似，我们只要挑选其中两个：一个用于 `创建`，一个用于 `更新`，根据业务商量就行了

- `put` 和 `patch` 的区别？\
  可以这么理解： `put` 是一个整体更新（我传一个对象，这个对象就要整体更新数据库的记录），而 `patch` 是部分更新（我传一个对象，我没传的部分你就给我留下来，传了的部分就覆盖之前的）

### 4. 构造

```ts
export class Http {
  constructor(baseUrl: string) {}
  get() {}
  post() {}
  patch() {}
  delete() {}
}
```

`constructor` 传一些基础的参数，比如：`BaseUR`

### 5. 实例

现在我们的底层请求还是用到了 Axios，所以我们还得有 Axios 这个 `实例`

```ts
export class Http {
  instance: AxiosInstance
  constructor(baseUrl: string) {}
  get() {}
  post() {}
  patch() {}
  delete() {}
}
```

### 6. 初始化方法

```ts
type JSONValue = string | number | null | boolean | JSONValue[] | { [key: string]: JSONValue }

export class Http {
  instance: AxiosInstance
  constructor(baseUrl: string) {
    this.instance = axios.create({
      baseURL,
      timeout: 10000
    })
  }

  get<R = unknown>(url: string, query?: Record<string, number | string>, config?: Omit<AxiosRequestConfig, 'params' | 'url' | 'method'>) {
    return this.instance.request<R>({ ...config, url, params: query, method: 'get' })
  }

  post<R = unknown>(url: string, data?: Record<string, JSONValue>, config?: Omit<AxiosRequestConfig, 'url' | 'data' | 'method'>) {
    return this.instance.request<R>({ ...config, url, data, method: 'post' })
  }

  patch<R = unknown>(url: string, data?: Record<string, JSONValue>, config?: Omit<AxiosRequestConfig, 'url' | 'data'>) {
    return this.instance.request<R>({ ...config, url, data, method: 'patch' })
  }

  delete<R = unknown>(url: string, query?: Record<string, string>, config?: Omit<AxiosRequestConfig, 'params'>) {
    return this.instance.request<R>({ ...config, url, params: query, method: 'delete' })
  }
}
```

如果我们以后想要换其他的请求库，只需要把 `instance`，换成其他的方法，就可以了，是不是减少了重构成本？

### 7. 统一的拦截处理

这里的统一拦截处理我们写在哪呢？\
答案是：写在哪都可以，由于我用到的接口不只有一个，所以我写在 class 外了

```ts
type JSONValue = string | number | null | boolean | JSONValue[] | { [key: string]: JSONValue }

export class Http {
  instance: AxiosInstance
  constructor(baseUrl: string) {
    this.instance = axios.create({
      baseURL,
      timeout: 10000
    })
  }

  get<R = unknown>(url: string, query?: Record<string, number | string>, config?: Omit<AxiosRequestConfig, 'params' | 'url' | 'method'>) {
    return this.instance.request<R>({ ...config, url, params: query, method: 'get' })
  }
  post<R = unknown>(url: string, data?: Record<string, JSONValue>, config?: Omit<AxiosRequestConfig, 'url' | 'data' | 'method'>) {
    return this.instance.request<R>({ ...config, url, data, method: 'post' })
  }
  patch<R = unknown>(url: string, data?: Record<string, JSONValue>, config?: Omit<AxiosRequestConfig, 'url' | 'data'>) {
    return this.instance.request<R>({ ...config, url, data, method: 'patch' })
  }
  delete<R = unknown>(url: string, query?: Record<string, string>, config?: Omit<AxiosRequestConfig, 'params'>) {
    return this.instance.request<R>({ ...config, url, params: query, method: 'delete' })
  }
}

export const http = new Http('/api/v1')

// 这里根据自己的业务需求封装
http.instance.interceptors.request.use(
  config => {},
  () => {}
)

http.instance.interceptors.response.use(
  response => {
    return response
  },
  error => {
    if (error.response) {
      const axiosError = error as AxiosError
      if (axiosError.response?.status === 429) {
        alert('你太频繁了')
      }
    }
    throw error
  }
)
```

# 使用

```ts
// 如果需要处理错误
const onError = () => {}
cosnt onClickSendValidationCode = async() => {
  const response = await http.post('/validationg_codes', { email })
    .catch(onError) // 失败
  console.log('成功') // 成功
}


// 如果在封装的时候处理了错误，我们可以这么写
cosnt onClickSendValidationCode = async() => {
  const response = await http.post('/validationg_codes', { email })
  console.log('成功')
}

```

是不是很方便？\
是不是很酷？

# 最后

> 其实，关于 `Axios 的二次封装`，网上已经有很多例子了，只不过这是我自己开发，就根据我的需求和思路进行封装

大家酌情选择吧！

感谢阅读，下次见 :)
