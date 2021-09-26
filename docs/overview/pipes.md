## 管道
管道是用`@Validation()`装饰器注释的类。管道应该实现IValidationPipe接口。

管道有两个典型的用例：

- 转换：将输入数据转换为所需的形式（例如，从字符串到整数）
- 验证：评估输入数据，如果有效，只需不变地传递它；否则，当数据不正确时抛出异常
__________

### 内置管道
Cirrus 带有开箱即用的管道：

- ValidationPipe
- ParseIntPipe (TODO)
- ParseFloatPipe (TODO)
- ParseBoolPipe  (TODO)
- ParseArrayPipe (TODO)
- ...

`ValidationPipe` 默认使用`class-validator`的数据校验方式。这个优秀的库允许您使用基于装饰器的验证。装饰器的功能非常强大，尤其是与 Cirrus 的 Pipe 功能相结合使用时。

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

__________

### 自定义管道
如前所述，您也可以构建自己的自定义管道。虽然 `Cirrus` 提供了强大的内置ValidationPipe，让我们看看自定义管道是如何构建的。

我们从一个简单的ValidationPipe. 使用`joi`的校验逻辑实现
```typescript
import {
    IValidationPipe,
    Validation,
    ValidationErrorException,
} from 'cirri/lib'
import * as Joi from 'Joi'

@Validation()
export class MyValidationPipe implements IValidationPipe {
    transform(data: any, schema: any): any {
        const { error, value } = Joi.validate(data, schema, {
            allowUnknown: true,
            abortEarly: true,
        })
        if (error) throw new ValidationErrorException(error)
        return value
    }
}
```
每个管道都必须实现`transform()`方法来履行`IValidationPipe`接口契约。这个方法有两个参数：

- `value`  数据
- `schema` 校验逻辑

 使用演示
```typescript
import { validate, Controller, IController } from 'cirri/lib'
import * as Joi from 'Joi'

@Controller()
export default class Demo implements IController {
    async index(data: { name: string; password: number }) {
        const schema = {
            name: Joi.string(),
            password: Joi.number().required(),
        }
        data = validate(data, schema)
        return data
    }
}

```
例如，假设路由为：

```
POST localhost:3000/demo body:{"name":"name"}
```
因为password为number类型且必传，所以会抛出这样的异常：
```
{
    "statusCode": 400,
    "code": "ValidationErrorException",
    "error": "\"password\" is required"
}
```
