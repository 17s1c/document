## 中间件
中间件是在路由处理程序 之前 调用的函数。 中间件函数可以访问请求和响应对象，以及应用程序请求响应周期中的 next() 中间件函数。 next() 中间件函数通常由名为 next 的变量表示。
![avatar](../image/Middlewares.png)
Cirrus 中间件实际上等价于 express 中间件。 下面是Express官方文档中所述的中间件功能：

中间件函数可以执行以下任务:

- 执行任何代码。
- 对请求和响应对象进行更改。
- 结束请求-响应周期。
- 调用堆栈中的下一个中间件函数。
- 如果当前的中间件函数没有结束请求-响应周期, 它必须调用 next() 将控制传递给下一个中间件函数。否则, 请求将被挂起。

__________
您可以在定义的类中使用 `@MiddleInter()` 装饰器实现自定义 Cirrus中间件。 

这个类应该实现 `IMiddleware` 接口的`use`方法。

`use`方法提供3个参数，分别是`Request`，`Response`，`next`

### 实现一个简单的中间件功能

```typescript
import { IMiddleware, MiddleInter } from 'cirri/lib'
import { Request, Response } from 'express'

@MiddleInter({ global:true })
export class APICallLoggerMiddleware implements IMiddleware {
    use(req: Request, _: Response, next): void {
        console.log('Request...');
        next();
    }
}

```
- 使用`App`提供的`init`方法加载中间件组件
```typescript
import { App } from '../../../packages/application'
import { APICallLoggerMiddleware } from './middlewares/apiCallLogger.middleware'

App.init(
    {
        middleware: [APICallLoggerMiddleware],
        ...
    },
).catch(err => console.error(err))
```


### 装饰器参数
`@MiddleInter()` 装饰器提供了2种拦截方式
- `global` 如果我们想一次将中间件绑定到每个注册的控制器中，我们可以传参global为true
```typescript
@MiddleInter({ global:true })
```
- `global` 如果我们只想对指定的控制器绑定，我们可以在controllers传参想要绑定的控制器

```typescript
@MiddleInter({ controllers: [Home,Demo] })
```

__________

### 依赖注入
中间件完全支持依赖注入。 就像提供者，它们能够注入属于同一模块的依赖项（通过 `constructor` ）。
