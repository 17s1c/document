## 控制器
### 控制器类
- 使用`@Controller()` 装饰器定义类为控制器，并需要实现`IController`接口的`index`方法
- 每个控制器为一个单独api，会根据类名生成一个`http` `POST`接口。
- `index`方法的`data`参数为请求`body`数据
```typescript
import { Controller, IController } from 'cirri/lib'

@Controller()
export default class Demo implements IController {
    async index(data) {
        ...
    }
}
```
```typescript
export interface IController {
    index(context: any): any;
}
```
### 路由
- 通过`app.module.ts`加载控制器组件
- 当启动项目时,为每个控制器生成对应的http路由，如`Demo`控制器会生成`POST` `/demo`接口
```typescript
import { App } from 'cirri/lib/application'
import Demo from './controllers/demo'

App.init(
    {
        controllers: [Demo],
        ...
    }
).catch(err => console.error(err))

```
