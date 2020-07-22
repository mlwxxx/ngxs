# 安装

## 安装

开始, 从npm安装包. 最新版本 \(3.x\) 支持 Angular/RxJS 6+, 如果你需要支持Angular5, 请用2.x版本.

```bash
npm install @ngxs/store --save

# 如果yarn是你的真爱
yarn add @ngxs/store
```

然后在 `app.module.ts`, 导入 `NgxsModule`:

```typescript
import { NgModule } from '@angular/core';
import { NgxsModule } from '@ngxs/store';

@NgModule({
  imports: [
    NgxsModule.forRoot([ZooState], {
      developmentMode: !environment.production
    })
  ]
})
export class AppModule {}
```

当您在导入中包含模块时, 你可以配置根存储 [options](../advanced/options.md). 如果您使用的是懒加载,你可以使用 `forFeature`选项并附相同参数的选项.

选项 比如 `developmentMode` 可以作为 `forRoot` 方法中的第二个参数传递给模块. 在开发模式下, 插件作者可以添加其他运行时（additional runtime） checks/etc 以增强开发人员体验。 切换到开发模式也会冻结您的store 使用 [deep-freeze-strict](https://www.npmjs.com/package/deep-freeze-strict) 模块.

在模块的根目录添加 `NgxsModule.forRoot([])` 很重要 即使您所有 states 都是 feature states.

## 开发构建

我们的持续集成服务器会在每次提交至master时运行所有测试，如果通过，它将向NPM发布新的开发版本，并使用@dev标记它。

这意味着，如果您想使用 `@ngxs/store` 或任何插件的最新功能，只需执行以下操作：

```bash
npm install @ngxs/store@dev --save
npm install @ngxs/logger-plugin@dev --save

# or if you are using yarn
yarn add @ngxs/store@dev
yarn add @ngxs/logger-plugin@dev

# of if you want to update multiple things at the same time
yarn add @ngxs/{store,logger-plugin,devtools-plugin}@dev
```

这将安装当前标记为“ @dev”的版本。 您的package.json文件将被锁定为该特定版本。

```javascript
{
  "dependencies": {
    "@ngxs/store": "3.0.0-dev.a0d076d"
  }
}
```

如果以后要再次更新到最新边缘，则必须再次运行上述命令。

