# Logger

一个简单的控制台日志插件，用于记录操作的处理过程。

## 安装

```bash
npm install @ngxs/logger-plugin --save

# or if you are using yarn
yarn add @ngxs/logger-plugin
```

## 用法


将 `NgxsLoggerPluginModule` 插件添加到您的根应用程序模块中：

```typescript
import { NgxsModule } from '@ngxs/store';
import { NgxsLoggerPluginModule } from '@ngxs/logger-plugin';

@NgModule({
  imports: [NgxsModule.forRoot([]), NgxsLoggerPluginModule.forRoot()]
})
export class AppModule {}
```

### 选项

该插件支持以下通过 `forRoot` 方法传递的选项：

* `logger`: 提供其他不同的记录器，对于记录到后端很有用。 默认为`console`。
* `collapsed`: 默认情况下是否折叠日志。默认为 `true`.
* `disabled`: 在生产环境禁用日志。 默认为 `false`.
* `filter`: 过滤要记录的动作。将动作和状态快照作为参数。  默认所有动作均返回 `true`.

```typescript
import { NgxsModule, getActionTypeFromInstance } from '@ngxs/store';
import { NgxsLoggerPluginModule } from '@ngxs/logger-plugin';
import { environment } from '../environments/environment';
import { customLogger } from './path/to/custom/logger';
import { SomeAction } from './path/to/some/action';

@NgModule({
  imports: [
    NgxsModule.forRoot([]),
    NgxsLoggerPluginModule.forRoot({
      // Use customLogger instead of console
      logger: customLogger,
      // Do not collapse log groups
      collapsed: false,
      // Do not log in production mode
      disabled: environment.production,
      // Do not log SomeAction
      filter: action => getActionTypeFromInstance(action) !== SomeAction.type
    })
  ]
})
export class AppModule {}
```


> `filter` 将状态快照作为第二个参数。对于一些特殊情况这很有用。但是，请注意以下事实：每个调度的动作调用都会调用它。 您可以考虑将带有记忆功能的方法用于过滤器，而不是简单的操作比较。

### 注意事项

您应该始终将日志作为配置中的最后一个插件。 例如，如果要在存储插件之类的插件之前添加logger，则初始状态将不会得到体现。

