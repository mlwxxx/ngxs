# installation

## 安装

To get started, install the package from npm. The latest version \(3.x\) supports Angular/RxJS 6+, if you want support for Angular 5, use version 2.x.

开始, 从npm安装包. 最新版本 \(3.x\) 支持 Angular/RxJS 6+, 如果你需要支持Angular5, 请用2.x版本.

```bash
npm install @ngxs/store --save

# 如果yarn是你的真爱
yarn add @ngxs/store
```

then in `app.module.ts`, import the `NgxsModule`:

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

When you include the module in the import, you can pass root stores along with [options](../advanced/options.md). If you are lazy loading, you can use the `forFeature` option with the same arguments.

当您在导入中包含模块时, 你可以配置根存储 [options](../advanced/options.md). 如果您使用的是懒加载,你可以使用 `forFeature`选项并附相同参数的选项.

Options such as `developmentMode` can be passed to the module as the second argument in the `forRoot` method. In development mode, plugin authors can add additional runtime checks/etc to enhance the developer experience. Switching to development mode will also freeze your store using [deep-freeze-strict](https://www.npmjs.com/package/deep-freeze-strict) module.

选项 比如 `developmentMode` 可以作为 `forRoot` 方法中的第二个参数传递给模块. 在开发模式下, 插件作者可以添加其他运行时（additional runtime） checks/etc 以增强开发人员体验。 切换到开发模式也会冻结您的store 使用 [deep-freeze-strict](https://www.npmjs.com/package/deep-freeze-strict) 模块.

It's important that you add `NgxsModule.forRoot([])` at the root of your module even if all of your states are feature states.

在模块的根目录添加 `NgxsModule.forRoot([])` 很重要 即使您所有 states 都是 feature states.

## Development Builds

Our continuous integration server runs all tests on every commit to master and if they pass it will publish a new development build to NPM and tag it with the @dev tag.

我们的持续集成服务器会在每次提交至master时运行所有测试，如果通过，它将向NPM发布新的开发版本，并使用@dev标记它。

This means that if you want the bleeding edge of `@ngxs/store` or any of the plugins you can simply do:

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

This will install the version currently tagged as `@dev`. Your package.json file will be locked to that specific version.

```javascript
{
  "dependencies": {
    "@ngxs/store": "3.0.0-dev.a0d076d"
  }
}
```

If you later want to again update to the bleeding edge, you will have to run the above command again.

如果以后要再次更新到最新边缘，则必须再次运行上述命令。

