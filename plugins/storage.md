# Storage

使用`localStorage`, `sessionStorage`或您希望的任何其他机制来支持存储。

## 安装

```bash
npm install @ngxs/storage-plugin --save

# or if you are using yarn
yarn add @ngxs/storage-plugin
```

## 使用

将 `NgxsStoragePluginModule` 导入到您的应用模块中，如下所示：

```typescript
import { NgxsModule } from '@ngxs/store';
import { NgxsStoragePluginModule } from '@ngxs/storage-plugin';

@NgModule({
  imports: [NgxsModule.forRoot([]), NgxsStoragePluginModule.forRoot()]
})
export class AppModule {}
```

建议在其他插件之前注册存储插件，以便这些插件可以获取初始状态。

### 选项

该插件具有以下选项：

* `key`: 要保留的状态名称. 您可以传递可以通过点表示法深度嵌套的字符串或字符串数组。 如果未提供，则使用, 所有状态都默认使用 `@@STATE` 。

* `storage`: 要使用的存储策略。 默认为LocalStorage，但您可以传递SessionStorage或任何实现了StorageEngine API的东西。
* `deserialize`: 自定义反序列化器。 默认为 `JSON.parse`
* `serialize`:  自定义序列化器。 默认为 `JSON.stringify`
* `migrations`: 迁移策略
* `beforeSerialize`: 在序列化之前执行的拦截器
* `afterDeserialize`: 在序列化之后执行的拦截器

### Key 选项

`key`选项用于确定哪些状态应保留在存储中。`key` 不能是随机字符串，它必须与您的状态名一致。 让我们看下面的例子：

```typescript
// novels.state.ts
@State<Novel[]>({
  name: 'novels',
  defaults: []
})
@Injectable()
export class NovelsState {}

// detectives.state.ts
@State<Detective[]>({
  name: 'detectives',
  defaults: []
})
@Injectable()
export class DetectivesState {}
```

为了保持所有状态，不需要提供`key`选项，因此只需编写即可：

```typescript
@NgModule({
  imports: [NgxsStoragePluginModule.forRoot()]
})
export class AppModule {}
```

但是，如果我们只想保存 `NovelsState` 怎么办？ 然后，我们需要将其名称传递给 `key` 选项：

```typescript
@NgModule({
  imports: [
    NgxsStoragePluginModule.forRoot({
      key: 'novels'
    })
  ]
})
export class AppModule {}
```

也可以提供一个与其名称相反的状态类：

```typescript
@NgModule({
  imports: [
    NgxsStoragePluginModule.forRoot({
      key: NovelsState
    })
  ]
})
export class AppModule {}
```

如果我们要保存 `NovelsState` 和 `DetectivesState` ：

```typescript
@NgModule({
  imports: [
    NgxsStoragePluginModule.forRoot({
      key: ['novels', 'detectives']
    })
  ]
})
export class AppModule {}
```

Or using state classes:

```typescript
@NgModule({
  imports: [
    NgxsStoragePluginModule.forRoot({
      key: [NovelsState, DetectivesState]
    })
  ]
})
export class AppModule {}
```

您甚至可以结合使用状态类和字符串：

```typescript
@NgModule({
  imports: [
    NgxsStoragePluginModule.forRoot({
      key: ['novels', DetectivesState]
    })
  ]
})
export class AppModule {}
```

这非常方便，在仅运行时\(runtime-only\)状态可以避免持久保存不应将保存到任何存储的。

### 自定义存储引擎

您可以通过实现 `StorageEngine` 接口来添加自己的存储引擎。

```typescript
import { NgxsStoragePluginModule, StorageEngine, STORAGE_ENGINE } from '@ngxs/storage-plugin';

export class MyStorageEngine implements StorageEngine {
  get length(): number {
    // Your logic here
  }

  getItem(key: string): any {
    // Your logic here
  }

  setItem(key: string, val: any): void {
    // Your logic here
  }

  removeItem(key: string): void {
    // Your logic here
  }

  clear(): void {
    // Your logic here
  }

  key(val: number): string {
    // Your logic here
  }
}

@NgModule({
  imports: [NgxsModule.forRoot([]), NgxsStoragePluginModule.forRoot()],
  providers: [
    {
      provide: STORAGE_ENGINE,
      useClass: MyStorageEngine
    }
  ]
})
export class MyModule {}
```

### 序列化拦截器

可以在状态被序列化或反序列化之前或之后定义自己的逻辑。

* beforeSerialize: 此选项可在序列化状态之前更改状态。
* afterSerialize: 序列化后，使用此选项可以更改状态。 例如，您可以使用它实例化一个具体的类。

```typescript
@NgModule({
  imports: [
    NgxsStoragePluginModule.forRoot({
      key: 'counter',
      beforeSerialize: (obj, key) => {
        if (key === 'counter') {
          return {
            count: obj.count < 10 ? obj.count : 10
          };
        }
        return obj;
      },
      afterDeserialize: (obj, key) => {
        if (key === 'counter') {
          return new CounterInfoStateModel(obj.count);
        }
        return obj;
      }
    })
  ]
})
export class AppModule {}
```

### 迁移

You can migrate data from one version to another during the startup of the store. Below is a strategy to migrate my state from `animals` to `newAnimals`.
您可以在存储启动期间将数据从一个版本迁移到另一个版本。 以下是将我的状态从 `animals` 迁移到 `newAnimals` 的策略。

```typescript
@NgModule({
  imports: [
    NgxsModule.forRoot([]),
    NgxsStoragePluginModule.forRoot({
      migrations: [
        {
          version: 1,
          key: 'zoo',
          versionKey: 'myVersion',
          migrate: state => {
            return {
              newAnimals: state.animals,
              version: 2 // Important to set this to the next version!
            };
          }
        }
      ]
    })
  ]
})
export class MyModule {}
```

在迁移策略中，我们定义：

* `version`: 我们正在迁移的版本
* `versionKey`: 版本标识符\(默认为'version'\)
* `migrate`: 接受状态并期望返回新状态的函数。
* `key`: 要迁移的项目的键。如果未指定，它将使用整个存储状态。

Note: Its important to specify the strategies in the order of which they should progress.
注意：重要的是要按其执行顺序指定策略。

