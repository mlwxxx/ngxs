# Store

store是一个全局状态管理器，可调度状态容器侦听的动作，并提供从全局状态中选择数据切片的方法。

## 创建 actions

一个action大概长成这样 `animal.actions.ts`.

```typescript
export class AddAnimal {
  static readonly type = '[Zoo] Add Animal';
  constructor(public name: string) {}
}
```

## 调度(Dispatching) actions

调度 actions, 您需要注入 `Store` 到你的组件/服务， 然后使用您希望触发的一个action(或一系列action的数组)调用 `dispatch` 方法。

```typescript
import { Store } from '@ngxs/store';
import { AddAnimal } from './animal.actions';

@Component({ ... })
export class ZooComponent {
  constructor(private store: Store) {}

  addAnimal(name: string) {
    this.store.dispatch(new AddAnimal(name));
  }
}
```

您还可以通过传递一个action数组来同时调度多个动作：

```typescript
this.store.dispatch([new AddAnimal('Panda'), new AddAnimal('Zebra')]);
```

假设执行动作后您要清除表单。 我们的`dispatch`方法 实际上返回了一个Observable，因此我们可以订阅它，并在成功后重新设置表单。

```typescript
import { Store } from '@ngxs/store';
import { AddAnimal } from './animal.actions';

@Component({ ... })
export class ZooComponent {

  constructor(private store: Store) {}

  addAnimal(name: string) {
    this.store.dispatch(new AddAnimal(name)).subscribe(() => this.form.reset());
  }

}
```

调度(dispatch)返回的Observable具有void类型,这是因为可以有多个状态(states)侦听同一`@ Action`，因此从这些操作(actions)上是不可能返回状态的，因为我们不知道他们是哪些states。


如果您需要在此之后获取状态，只需在链中使用`@ Select`即可，例如：

```typescript
import { Store, Select } from '@ngxs/store';
import { Observable } from 'rxjs';
import { withLatestFrom } from 'rxjs/operators';
import { AddAnimal } from './animal.actions';

@Component({ ... })
export class ZooComponent {

  @Select(state => state.animals) animals$: Observable<any>;

  constructor(private store: Store) {}

  addAnimal(name: string) {
    this.store
      .dispatch(new AddAnimal(name))
      .pipe(withLatestFrom(this.animals$))
      .subscribe(([_, animals]) => {
        // do something with animals
        this.form.reset();
      });
  }

}
```

## 快照(Snapshots)


您可以通过调用 `store.snapshot()`获取状态的快照.它将返回该时间点的整个存储值。
## 选择状态(State)

转到[select](select.md) 页面查看有关如何使用存储区选择数据的详细信息。

## 重启(Reset)

在某些情况下，您需要能够在不触发任何操作或生命周期挂钩的情况下完全重置状态。 一个例子是当我们进行时间旅行时的redux devtools插件。 另一个例子是当我们进行单元测试时，需要状态成为隔离测试的特定值。

`store.reset(myNewStateObject)` 会将整个状态重置为传递的参数，而不触发任何操作或生命周期事件。

警告：如果使用不当，可能会导致意想不到的副作用，请谨慎服用 :)

