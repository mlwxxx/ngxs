# 取消

如果您有异步动作，则如果该操作已被再次调度，则可能要取消先前的Observable。 这对于取消先前的请求(前面的输入)很有用。

## 简单

简单场景，我们可以使用 `cancelUncompleted` 装饰器选项。

```typescript
import { Injectable } from '@angular/core';
import { State, Action } from '@ngxs/store';

@State<ZooStateModel>({
  defaults: {
    animals: []
  }
})
@Injectable()
export class ZooState {
  constructor(private animalService: AnimalService, private actions$: Actions) {}

  @Action(FeedAnimals, { cancelUncompleted: true })
  get(ctx: StateContext<ZooStateModel>, action: FeedAnimals) {
    return this.animalService.get(action.payload).pipe(
      tap((res) => ctx.setState(res))
    ));
  }
}
```

## 高级

对于更高级的情况，我们可以使用普通的Rx运算符。

```typescript
import { Injectable } from '@angular/core';
import { State, Action, Actions, ofAction } from '@ngxs/store';
import { tap } from 'rxjs/operators';

@State<ZooStateModel>({
  defaults: {
    animals: []
  }
})
@Injectable()
export class ZooState {
  constructor(private animalService: AnimalService, private actions$: Actions) {}

  @Action(FeedAnimals)
  get(ctx: StateContext<ZooStateModel>, action: FeedAnimals) {
    return this.animalService.get(action.payload).pipe(
      tap((res) => ctx.setState(res)),
      takeUntil(this.actions$.pipe(ofAction(RemoveTodo)))
    ));
  }
}
```

