## 安装和初始化
- 安装命令行工具
```bash
$ yarn global add cirrus_cli
```
- 初始化
```bash
$ cirrus init my_project
$ cd my_project            # 进入项目根目录

# 运行
$ yarn start

# 运行生产环境
$ yarn build
$ yarn start:pro
```

## 初次相识

### 目录结构

```
.
├── src                          
│   ├── config                                    配置文件
│   │   └── config.ts                   
│   ├── controllers                               控制器层目录
│   │   └── demo.ts         
│   ├── dtos                                      dto层目录
│   │   └── demoPostReq.dto.ts 
│   │   └── demoPostRes.dto.ts 
│   ├── filters                                   异常过滤器目录
│   │   └── httpException.filter.ts 
│   ├── middlewares                               中间件层目录
│   │   └── apiCallLogger.middleware.ts 
│   ├── model                                     model层目录
│   │   └── user.model.ts                         
│   ├── pipes                                     管道层目录
│   │   └── validation.pipe.ts 
│   ├── service                                   service层目录
│   │   └── demo.service.ts 
│   ├── app.module.ts                             程序初始入口模块
├── .cirrus                                       框架配置文件     
├── .env                                          全局环境变量
├── .prettierrc                                   代码格式配置文件
├── .vscode                                       vscode配置文件
├── package.json                       
├── public                                        静态资源                  
├── sdk                                           浏览器端 SDK             
├── tsconfig.json                                 typescript配置文件
├── tslint.json                                   代码规范配置文件             
.


```

### 第一个 API
#### 服务端如何实现

- 创建Demo.ts文件，使用 `@Controller()` 装饰器定义一个基本的控制器，继承IController接口实现index方法
- 通过`validate`方法校验请求数据是否正确，validate校验逻辑默认使用[class-validator](https://github.com/typestack/class-validator)
也可以自定义校验逻辑详细见[Pipes]() 
```typescript
import { validate, Controller, IController } from 'cirri/lib'
import { IsNumber, IsString } from 'class-validator'

export class DemoLoginPostReqDto {
    @IsString()
    readonly name: string
    @IsNumber()
    readonly password: number
}

@Controller()
export default class Demo implements IController {
    async index(data: DemoLoginPostReqDto) {
        data = validate(data, DemoLoginPostReqDto)
        return data
    }
}
```
#### 客户端如何调用
- 使用命令行工具生成浏览器端SDK 
```bash
$ cirrus_cli sdk
```
- 页面导入生成的sdk,调用后端api(代码示例)
```html
# demo.html
<html lang="en">
    <body>
        <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
        <script type="module" src="demo.js"></script>
    </body>
</html>
```
```typescript
// demo.js
import { Demo } from 'sdk/cirrusSdk.js'
Demo({
    name: 'name',
    password: 123,
})
    .then(function(data) {
        alert(JSON.stringify(data))
    })
    .catch(function(err) {
        console.log(err)
    })

```
- (AOP 切面编程，对请求流的处理，见 [Middleware]())

### 操作数据库
- `cirrus`框架集成了`TypeORM`框架，支持SQL和NoSQL数据库。`cirrus` 使用TypeORM是因为它是 TypeScript 中最成熟的对象关系映射器( ORM )。因为它是用 TypeScript 编写的，所以可以很好地与 `cirrus` 框架集成。

- `TypeORM` 提供了对许多关系数据库的支持，比如 `PostgreSQL` 、`Oracle`，甚至像 `MongoDB`这样的 NoSQL 数据库，下面代码演示如何使用流行的Mysql 
##### 1. 首先需要添加mysql配置信息
```bash
# .env
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DBNAME=demo
MYSQL_USERNAME=root
MYSQL_PASSWORD=12345678
```
```typescript
// config.ts
import { AppConfig } from 'cirri/lib'
export const config: AppConfig = {
    port: Number(process.env.PORT),
    dbOptions: {
        type: 'mysql',
        host: process.env.MYSQL_HOST,
        port: Number(process.env.MYSQL_PORT),
        username: process.env.MYSQL_USERNAME,
        password: process.env.MYSQL_PASSWORD,
        database: process.env.MYSQL_DBNAME,
        logging: true,
        synchronize: true,
    },
}
```
##### 2. 定义UserModel实体类
- `BaseModel`定义了基本的数据字段包括`id` `uuid` `createdAt` `updatedAt` 等
```typescript
import { BaseModel } from 'cirri/lib'
import { Column, Entity } from 'typeorm'

@Entity('users')
export default class UserModel extends BaseModel {
    @Column()
    name: string

    @Column()
    password: string
}
```

##### 3. 控制层对User保存操作
- 使用`InjectModel`装饰器注入UserModel实体依赖关系
- 实现User保存到数据库逻辑
```typescript
import { InjectModel, Controller, IController } from 'cirri/lib'
import { Repository } from 'typeorm'
import UserModel from './user.model'

@Controller()
export default class Demo implements IController {
    constructor(
        @InjectModel(UserModel)
        private userRepository: Repository<UserModel>,
    ) {}
    async index(user) {
        let userModel = new UserModel()
        userModel = Object.assign(user, userModel)
        return this.userRepository.save(userModel)
    }
}

```
##### 4. 加载组件
- 使用`App`提供的`init`方法加载组件
```typescript
import { App } from 'cirri/lib/application'
import Demo from './controllers/demo'
import UserModel from './model/user.model'
import { config } from './config/config'

App.init(
    {
        controllers: [ Demo],
        model: [UserModel],
        ...
    },
    config,
).catch(err => console.error(err))

```

### 调用 RPC 以及微服务

### 复杂逻辑

将复杂功能拆解，变成功能边界清晰的服务。对我们第一个应用进行代码重构

## 下一步

- [打包和部署](./deploy.md)
- [功能调试](./debug.md)
- [Controller](./controller.md)
- [Provider](./provider.md)
- [Middleware](./middleware.md)
- [Pipes](./pipes.md)
- [filter](./filter.md)
- [Configuration](./configuration.md)
- [Template]()
