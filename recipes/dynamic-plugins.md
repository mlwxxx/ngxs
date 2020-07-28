# 动态插件


与生产或其他构建目标相比，Angular提供了为开发加载不同环境文件的功能。 当涉及仅开发包时，我们可以使用它来改善我们的应用程序捆绑。 在NGXS中，主要仅对开发模式有用的软件包是 `@ngxs/devtools-plugin` 和 `@ngxs/logger-plugin`。 通常，您只想在开发过程中而不是在生产中使用这些软件包。

我们来看下面的代码：

```typescript
// environment.ts
import { NgxsLoggerPluginModule } from '@ngxs/logger-plugin';
import { NgxsReduxDevtoolsPluginModule } from '@ngxs/devtools-plugin';

export const environment = {
  production: false,
  plugins: [NgxsLoggerPluginModule.forRoot(), NgxsReduxDevtoolsPluginModule.forRoot()]
};
```

这意味着仅当Angular使用 `environment.ts` 文件时才使用这些插件，但是在生产版本中它将被 `environment.prod.ts` 文件\(或您使用的任何其他配置\)替换。 如果您已经发现 `environment.prod.ts` 文件将包含等于空数组的 `plugins` 属性，代码将文件下面这样：

```typescript
// environment.prod.ts
export const environment = {
  production: true,
  plugins: []
};
```

我们剩下要做的就是导入环境文件，并在 `AppModule` 导入中引用 `plugins` 属性：

```typescript
import { NgxsModule } from '@ngxs/store';

import { environment } from '../environments/environment';

@NgModule({
  imports: [NgxsModule.forRoot([]), environment.plugins]
})
export class AppModule {}
```

这样将减小您的产品包大小，因为仅在开发期间才需要这些软件包。

