## 提供者

- 服务，复杂业务逻辑和流程，可以复用的功能服务
- Job，后台任务，定时任务
- 云产品，各大云厂商提供的云产品能力
__________

Providers 是 Cirrus 的基本概念。许多基本的 Cirrus 类可能被视为 provider - service, repository, factory, helper 等等。 他们都可以通过 constructor 注入依赖关系。 这意味着对象可以彼此创建各种关系，并且“连接”对象实例的功能在很大程度上可以委托给 Cirrus。 Provider 只是一个用 @Provider() 装饰器注释的类。
#### 自定义提供者
- 自定义Dao服务
```typescript
import { InjectModel, Provider } from 'cirri/lib'
import { Repository } from 'typeorm'

import UserModel from '../model/user.model'

@Provider()
export default class DemoDao {
    constructor(
        @InjectModel(UserModel)
        private userRepository: Repository<UserModel>,
    ) {}

    async findOne() {
        return await this.userRepository.find()
    }
}
```
-  自定义Service服务

 `DemoDao`可以通过 constructor 注入依赖关系
```typescript
import { Provider } from 'cirri/lib'
import { DemoDao } from './DemoDao'

@Provider()
export default class DemoService {
    constructor(
        private demoDao: DemoDao,
    ) {}

    async query() {
        return await this.demoDao.findOne()
    }
}
```
#### 内置提供者
- 使用Cirrus内置的loggerService服务
```typescript
import { Provider,LoggerService } from 'cirri/lib'

@Provider()
export default class DemoService {
    constructor(
        private readonly loggerService: LoggerService
    ) {}

    async query() {
        this.loggerService.info('日志输出')
    }
}
```
#### 依赖注入

Cirrus 是围绕通常称为依赖注入的强大设计模式构建的。

在 Cirrus 中，由于 TypeScript 功能，管理依赖项非常容易，因为它们仅按类型解析。在下面的示例中，Cirrus 将demoService通过创建并返回一个实例DemoService（或者，在单例的正常情况下，如果已经在其他地方请求过，则返回现有实例）来解决 。此依赖项已解析并传递给控制器​​的构造函数（或分配给指定的属性）：
```
constructor(
    private demoService: DemoService,
    private demoDao: DemoDao,
    private loggerService: LoggerService,
) {}

```
__________
### 各大云厂商的云产品

Providers还提供了各大云厂商的云产品能力，如OSS,cache,sls,mqtt,短信服务等等（待完成）

- OSS服务
```typescript
constructor(
    @Inject('alibaba')
    private ossService: OssService,
) {}

```
- mqtt服务
```typescript
constructor(
    @Inject('aws')
    private mqttService: MqttService,
) {}

```
