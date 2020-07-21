# 动作(Actions)

Actions 可以将操作视为应触发某些事件发生的命令，也可以将其视为已发生某事的结果事件。

每个动作都包含一个“type”字段，这是它们的唯一标识符。

## 内部Actions

库的内部触发了两个Actions：

1. @@INIT - store being initialized, before all the [ngxsOnInit Life-cycle](../advanced/life-cycle.md) events.
2. @@UPDATE\_STATE - a new [lazy-loaded state](../advanced/lazy.md) being added to the store.

## 简单 Action

Let's say we want to update the status of whether the animals have been fed in our Zoo. We would describe a class like:
假设我们要更新一个状态，它表示某动物是否已在我们的动物园饲养。 我们将这样描述一个类：

```typescript
export class FeedAnimals {
  static readonly type = '[Zoo] Feed Animals';
}
```

然后在我我们的state类, we will listen to this action and mutate our state, in this case flipping a boolean flag.
然后在我我们的state类, 我们将监听取这个action并改变我们的state,在这种情况中就是翻转布尔值标志。

## 带有元数组的Action

通常，您需要执行一项操作才能将一些数据与之关联。 在这里，有一个action应该触发————给斑马喂干草。

```typescript
export class FeedZebra {
  static readonly type = '[Zoo] Feed Zebra';
  constructor(public name: string, public hayAmount: number) {}
}
```

动作类的“name”字段代表我们将喂食的斑马的名字。 `hayAmount`告诉我们斑马应该得到多少公斤干草。

## 调度动作(Action)

转到 [Store](store.md) 查看有关如何调度动作的文档。

## 您应该如何命名自己的actions?

### 指令

Commands are actions that tell your app to do something. They are usually triggered by user events such as clicking on a button, or selecting something.

Names should contain three parts:

* A context as to where the command came from, `[User API]`, `[Product Page]`, `[Dashboard Page]`.
* A verb describing what we want to do with the entity.
* The entity we are acting upon, `User`, `Card`, `Project`.

Examples:

* `[User API] GetUser`
* `[Product Page] AddItemToCart`
* `[Dashboard Page] ArchiveProject`

### Event examples

Events are actions that have already happened and we now need to react to them.

The same naming conventions apply as commands, but they should always be in the past tense.

By using `API` in the context part of the action name we know that this event was fired because of an async action to an API.

Actions are normally dispatched from container components such as router pages. By having explicit actions for each page, it's also easier to track where an event came from.

Examples:

* \[User API\] GetUserSuccess
* \[Project API\] ProjectUpdateFailed
* \[User Details Page\] PasswordChanged
* \[Project Stars Component\] StarsUpdated

A great video on the topic is [Good Action Hygiene by Mike Ryan](https://www.youtube.com/watch?v=JmnsEvoy-gY) It's for NgRx, but the same naming conventions apply to NGXS.

## Group your actions

Don't suffix your actions:

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

here we group similar actions into the `Todo` namespace. In this case just import namespace instead of multiple action classes in same file.

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

