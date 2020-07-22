# 动作(Actions)

Actions 可以将操作视为应触发某些事件发生的命令，也可以将其视为已发生某事的结果事件。

每个动作都包含一个“type”字段，这是它们的唯一标识符。

## 内部动作(Internal Actions)

库的内部触发了两个Actions：

1. @@INIT - store正在初始化, 在[ngxsOnInit生命周期](../advanced/life-cycle.md) 之前.
2. @@UPDATE\_STATE - 一个新的[延迟加载state](../advanced/lazy.md)被添加到store.

## 简单动作(Simple Action)

假设我们要更新一个状态，它表示某动物是否已在我们的动物园饲养。 我们将这样描述一个类：

```typescript
export class FeedAnimals {
  static readonly type = '[Zoo] Feed Animals';
}
```

然后在我我们的state类, 我们将监听取这个action并改变我们的state,在这种情况中就是翻转布尔值标志。

## 带有元数组的动作(Actions with Metadata)

通常，您需要执行一项操作才能将一些数据与之关联。 在这里，有一个action应该触发————给斑马喂干草。

```typescript
export class FeedZebra {
  static readonly type = '[Zoo] Feed Zebra';
  constructor(public name: string, public hayAmount: number) {}
}
```

动作类的“name”字段代表我们将喂食的斑马的名字。 `hayAmount`告诉我们斑马应该得到多少公斤干草。

## 调度动作\(Dispatching Actions\)

转到 [Store](store.md) 查看有关如何调度动作的文档。

## 您应该如何命名自己的actions?

### 命令\(Commands\)

命令是action,它告诉您的应用执行某些任务。 它们通常是由用户事件触发的，例如单击按钮或选择某些东西。 名称应包含三个部分:

* 一个描述命令来自何处的上下文\(context\), `[User API]`, `[Product Page]`, `[Dashboard Page]`.
* 描述我们要对实体做什么的动词.
* 我们所操作的实体, `User`, `Card`, `Project`.

示例:

* `[User API] GetUser`
* `[Product Page] AddItemToCart`
* `[Dashboard Page] ArchiveProject`

### 事件示例

事件是已经发生的、我们现在需要对它们做出反应的动作\(actions\)。

相同的命名约定适用于命令\(Commands\)，但它们应始终使用过去时。

在上下文动作名称部分使用`API`，让我们知道此事件是由一个作用于API的异步动作而触发的。

通常动作\(Action\)由容器组件（如路由器页面）分派。每个页面拥有明确的动作\(actions\)，跟踪事件的来源也变得更加容易。

示例:

* \[User API\] GetUserSuccess
* \[Project API\] ProjectUpdateFailed
* \[User Details Page\] PasswordChanged
* \[Project Stars Component\] StarsUpdated

有一个关于这个主题的精彩视频 [Good Action Hygiene by Mike Ryan](https://www.youtube.com/watch?v=JmnsEvoy-gY) 它是基于NgRx的, 但是相同的命名约定也适用于NGXS。

## 分组动作\(actions\)

不要用后缀命名你的动作\(actions\):

```typescript
export class AddTodo {
  static readonly type = '[Todo] Add';
  constructor(public payload: any) {}
}

export class EditTodo {
  static readonly type = '[Todo] Edit';
  constructor(public payload: any) {}
}

export class FetchAllTodos {
  static readonly type = '[Todo] Fetch All';
}

export class DeleteTodo {
  static readonly type = '[Todo] Delete';
  constructor(public id: number) {}
}
```

在这里，我们将类似的动作分组到`Todo`命名空间中。 在这种情况下，只需导入名称空间，而不是在同一文件中导入多个动作类。

```typescript
export namespace Todo {
  export class Add {
    static readonly type = '[Todo] Add';
    constructor(public payload: any) {}
  }

  export class Edit {
    static readonly type = '[Todo] Edit';
    constructor(public payload: any) {}
  }

  export class FetchAll {
    static readonly type = '[Todo] Fetch All';
  }

  export class Delete {
    static readonly type = '[Todo] Delete';
    constructor(public id: number) {}
  }
}
```

