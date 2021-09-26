## 配置

#### 环境变量配置
应用程序通常在不同的环境中运行。根据环境的不同，应该使用不同的配置设置。例如，通常本地环境依赖于本地 DB实例。生产环境将使用生产的 DB实例。由于配置变量会更改，所以最佳实践是将配置变量.env文件中。

在 Cirrus 中 `AppConfig`通过 process.env会自动加载 .env 文件的配置信息
```
.env
```
```
NODE_ENV=local
PORT=8080
SERVICE_URL=http://localhost:8080

## mysql
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DBNAME=demo
MYSQL_USERNAME=root
MYSQL_PASSWORD=12345678
```
```
config.ts
```
```typescript
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
.env配置说明

- NODE_ENV - 运行环境
- PORT- 端口
- SERVICE_URL- 用于生成浏览器端SDK

appConfig配置说明
- port - 端口
- dbOptions 数据库连接配置 详细见[typeorm](https://typeorm.io/#/connection-options)

------
#### 组件配置
```
app.module.ts
```
```typescript
import { App } from 'cirri/lib/application'

App.init(
    {
        controllers: [Demo],
        validationPipe: MyValidationPipe,
        httpExceptionFilter: MyHttpExceptionFilter,
        middleware: [APICallLoggerMiddleware],
        providers: [DemoService],
        enableCors: true,
        model: [UserModel],
    },
    config,
).catch(err => console.error(err))

```
组件配置说明

- controllers - 控制器组件
- validationPipe- 管道组件
- httpExceptionFilter- 异常过滤器组件
- middleware- 中间件
- providers- 提供者组件
- enableCors- 跨域配置信息 详细见[cors](https://expressjs.com/en/resources/middleware/cors.html)
- model- 模型实体
- config- 环境变量配置
