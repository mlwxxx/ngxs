# 介绍

我们够谈谈插件。 与Redux的meta reducers类似，我们有一个插件界面，该界面可让您为您的状态构建全局插件。

您所要做的就是为`NGXS_PLUGINS`令牌提供一个类。 如果您的插件具有与其相关联的选项，建议您定义一个注入令牌，然后在您的模块上定义一个`forRoot`方法。

让我们看一个日志的基本示例：

```typescript
import { Injectable, Inject, NgModule } from '@angular/core';
import { NgxsPlugin, NGXS_PLUGINS } from '@ngxs/store';

export const NGXS_LOGGER_PLUGIN_OPTIONS = new InjectionToken('NGXS_LOGGER_PLUGIN_OPTIONS');

@Injectable()
export class LoggerPlugin implements NgxsPlugin {
  constructor(@Inject(NGXS_LOGGER_PLUGIN_OPTIONS) private options: any) {}

  handle(state, action, next) {
    console.log('Action started!', state);
    return next(state, action).pipe(
      tap(result => {
        console.log('Action happened!', result);
      })
    );
  }
}

@NgModule()
export class NgxsLoggerPluginModule {
  static forRoot(config?: any): ModuleWithProviders {
    return {
      ngModule: NgxsLoggerPluginModule,
      providers: [
        {
          provide: NGXS_PLUGINS,
          useClass: LoggerPlugin,
          multi: true
        },
        {
          provide: NGXS_LOGGER_PLUGIN_OPTIONS,
          useValue: config
        }
      ]
    };
  }
}
```

您也可以对插件使用纯函数。 上面例子用纯函数实现如下：

```typescript
export function logPlugin(state, action, next) {
  console.log('Action started!', state);
  return next(state, action).pipe(tap(result) => {
    console.log('Action happened!', result);
  });
}
```


注意：提供纯函数时，请确保使用 `useValue` 而不是 `useClass`。

要向NGXS注册插件，请在模块中导入插件模块，并传递如下插件选项(可选)：

```typescript
@NgModule({
  imports: [NgxsModule.forRoot([ZooStore]), NgxsLoggerPluginModule.forRoot({})]
})
export class MyModule {}
```


该方法也适用于 `forFeature`。

