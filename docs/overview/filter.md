## 异常过滤器
Cirrus 带有一个内置的异常层，负责处理整个应用程序中所有未处理的异常。当您的应用程序代码未处理异常时，该层会捕获该异常，然后该层会自动发送用户友好响应。

开箱即用，此操作由内置的全局异常过滤器执行，该过滤器处理类型HttpException\的异常。当无法识别异常时，内置异常过滤器会生成以下默认 JSON 响应：
```json
{
    "code": 500,
    "error": "Internal server error"
}
```
__________

### 内置 HTTP 异常
__________

Cirrus 提供了一组从 BaseException 继承的标准异常HttpException。许多最常见的 HTTP 异常：
- ValidationErrorException
- BadRequestErrorException
- UnauthorizedErrorException
- NotFoundErrorException
- ForbiddenErrorException
- NotAcceptableErrorException
- RequestTimeoutErrorException
- ConflictErrorException
- GoneErrorException
- HttpVersionNotSupportedErrorException
- PayloadTooLargeErrorException
- UnsupportedMediaTypeErrorException
- UnprocessableEntityErrorException
- ...

你也可以自定义异常
```typescript
import { BaseException } from 'cirri/lib'

export class CustomErrorException extends BaseException {
    constructor(err = null) {
        super(500, 'CustomErrorException', err)
    }
}
```
使用
```typescript
async findAll() {
  throw new CustomErrorException('custom error');
}
```
### 自定义异常过滤器
__________
虽然Cirrus自带异常过滤器可以为您自动处理许多情况，但有时您可能希望对异常层拥有完全控制权，例如，您可能希望基于某些动态因素添加日志记录或使用不同的 JSON 模式。 异常过滤器正是为此目的而设计的。 它们使您可以控制精确的控制流以及将响应的内容发送回客户端。

- `@Exception()`定义类为异常过滤器，并实现`IExceptionFilter`接口的`catch`方法

```typescript
import { Exception, IExceptionFilter } from 'cirri/lib'

@Exception()
export class MyHttpExceptionFilter implements IExceptionFilter {
    catch(err, req, res, next): void {
        console.error(err)
        res.status(err?.status || 500)
        res.send({
            code: err?.code,
            error: err?.message,
        })
    }
}

```
