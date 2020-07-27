# 状态(State)

状态\(States\)是一个用来定义状态容器的类。

## 定义一个状态

States是一个带有装饰器的类，用于描述元数据\(metadata\)和动作\(action\)映射。为了定义一个状态容器，让我们创建一个ES2015类，并用`State`装饰器装饰它。

```typescript
import { Injectable } from '@angular/core';
import { State } from '@ngxs/store';

@State<string[]>({
  name: 'animals',
  defaults: []
})
@Injectable()
export class AnimalsState {}
```

在状态装饰器中，我们定义有关状态\(State\)的一些元数据。 这些选项包括：

* `name`: 状态片的名称。 注意：名称是必填参数，并且对于整个应用程序必须是唯一的。

  名称必须是对象属性安全的, \(如果不能包含破折号、点等等\).

* `defaults`: 此状态切片默认值的集合\(对象/数组\)。
* `children`: 子状态关联.

我们的states也可以依赖注入。 并且是自动连接的，因此您要做的就是将依赖项注入到构造函数中。

```typescript
@State<ZooStateModel>({
  name: 'zoo',
  defaults: {
    feed: false
  }
})
@Injectable()
export class ZooState {
  constructor(private zooService: ZooService) {}
}
```

## \(可选的\) 定义状态令牌

（可选）, 您可以使用一个状态令牌\(state token\)替换你的状态名`name`

```typescript
const ZOO_STATE_TOKEN = new StateToken<ZooStateModel>('zoo');

@State({
  name: ZOO_STATE_TOKEN,
  defaults: {
    feed: false
  }
})
@Injectable()
export class ZooState {
  constructor(private zooService: ZooService) {}
}
```

这种稍微高级的方法具有一些优点，你可转到[State Token](../advanced/token.md) 部分中了解更多信息。

## 定义动作\(Actions\)

states通过一个@Action装饰器来侦听动作。 动作装饰器接受一个动作类或动作类数组。

### 简单动作\(Actions\)

Let's define a state that will listen to a `FeedAnimals` action to toggle whether the animals have been fed: 让我们定义一个状态，该状态将侦听`FeedAnimals`动作来切换动物是否已喂食：

```typescript
import { Injectable } from '@angular/core';
import { State, Action, StateContext } from '@ngxs/store';

export class FeedAnimals {
  static readonly type = '[Zoo] FeedAnimals';
}

export interface ZooStateModel {
  feed: boolean;
}

@State<ZooStateModel>({
  name: 'zoo',
  defaults: {
    feed: false
  }
})
@Injectable()
export class ZooState {
  @Action(FeedAnimals)
  feedAnimals(ctx: StateContext<ZooStateModel>) {
    const state = ctx.getState();
    ctx.setState({
      ...state,
      feed: !state.feed
    });
  }
}
```

`feedAnimals`函数具有一个名为ctx的参数，其类型为 `StateContext<ZooStateModel>`.此上下文状态具有切片指针和公开用于设置状态的函数。 需要特别注意的是，每次访问时，`getState()`方法将始终从全局存储返回最新的状态片。 这样可以确保当我们执行异步操作时，状态始终是最新的。 如果需要快照，则始终可以在方法中克隆状态\(state\)。

### 具有有效载荷的动作(Actions with a payload)

动作也可以传递与动作有关的元数据。 假设我们要传递每个斑马需要多少干草和胡萝卜。

```typescript
import { Injectable } from '@angular/core';
import { State, Action, StateContext } from '@ngxs/store';

// This is an interface that is part of your domain model
export interface ZebraFood {
  name: string;
  hay: number;
  carrots: number;
}

// naming your action metadata explicitly makes it easier to understand what the action
// is for and makes debugging easier.
export class FeedZebra {
  static readonly type = '[Zoo] FeedZebra';
  constructor(public zebraToFeed: ZebraFood) {}
}

export interface ZooStateModel {
  zebraFood: ZebraFood[];
}

@State<ZooStateModel>({
  name: 'zoo',
  defaults: {
    zebraFood: []
  }
})
@Injectable()
export class ZooState {
  @Action(FeedZebra)
  feedZebra(ctx: StateContext<ZooStateModel>, action: FeedZebra) {
    const state = ctx.getState();
    ctx.setState({
      ...state,
      zebraFood: [
        ...state.zebraFood,
        // this is the new ZebraFood instance that we add to the state
        action.zebraToFeed
      ]
    });
  }
}
```

在这个示例中，我们有第二个参数表示动作，然后对其进行解构以提取名称，干草和胡萝卜，然后使用其更新状态。

还有一个快捷的函数 `patchState`，使更新状态变得更加容易。在这种情况下，您仅向其传递要在状态上更新的属性，然后它将处理其余的属性。 上面的功能可以简化为：

```typescript
@Action(FeedZebra)
feedZebra(ctx: StateContext<ZooStateModel>, action: FeedZebra) {
  const state = ctx.getState();
  ctx.patchState({
    zebraFood: [
      ...state.zebraFood,
      action.zebraToFeed,
    ]
  });
}
```

The `setState` function can also be called with a function which will be given the existing state and should return the new state. All immutability concerns need to be honoured by this function. `setState`函数也可以通过将被赋予现有状态并应返回新状态的函数来调用。 此函数必须满足所有不变性的考虑。

为了进行比较，以下是两种可以调用 `setState`函数的方式... 使用新构造的状态\(state\)值：

```typescript
@Action(MyAction)
public addValue(ctx: StateContext, { payload }: MyAction) {
  ctx.setState({ ...ctx.getState(), value: payload  });
}
```

使用返回新状态值的函数：

```typescript
@Action(MyAction)
public addValue(ctx: StateContext, { payload }: MyAction) {
  ctx.setState((state) => ({ ...state, value: payload }));
}
```

您可能会问 _"这有什么价值？"_ 好的，它为将不可变的更新重构为`state operators`打开了大门，因此您的代码可以变得更具声明性，而不再是命令性的。 我们将很快添加一些标准的`state operators`，您将可以使用它们来表示对状态的更新。请在此处关注该问题以进行更新: [https://github.com/ngxs/store/issues/545](https://github.com/ngxs/store/issues/545)

再举一个例子，您可以使用 [immer](https://github.com/mweststrate/immer) 之类的库，该库可以为您处理不变性更新，并提供通过直接修改草稿对象来表达您不变性更新的另一种方式 。 我们可以使用此外部库，因为它通过其咖喱的`produce`函数支持与`state operators`相同的签名。 这是上面以这种方式表达的示例：

```typescript
import produce from 'immer';

// in class ZooState ...
@Action(FeedZebra)
feedZebra(ctx: StateContext<ZooStateModel>, action: FeedZebra) {
  ctx.setState(produce((draft) => {
    draft.zebraFood.push(action.zebraToFeed);
  }));
}
```

在这里，仅使用单个参数调用 `immer` 库中的`produce`函数，以便它返回其[curried form](https://github.com/mweststrate/immer#currying)，它将采用一个值， 返回一个新值，并应用所有表示的更改。

这种方法还可以允许创建命名良好的帮助程序功能，这些功能可以在需要相同类型更新的处理程序之间共享。 上面的示例可以重构为：

```typescript
// in class ZooState ...
@Action(FeedZebra)
feedZebra(ctx: StateContext<ZooStateModel>, action: FeedZebra) {
  ctx.setState(addToZebraFood(action.zebraToFeed));
}

// defined elsewhere
import produce from 'immer';

function addToZebraFood(itemToAdd) {
  return produce((draft) => {
    draft.zebraFood.push(itemToAdd);
  });
}
```

### 异步动作

动作\(Actions\)可以执行异步操作并在操作后更新状态。

通常，在Redux中，您的操作是纯函数，并且您还可以使用其他系统（例如saga或effect）来执行这些操作，并将其他动作分派回您的状态以进行更改。 这有一些原因，但是在大多数情况下，它可能是多余的，只需添加样板即可。 很棒的是，我们让您可以根据自己的需求灵活地做出决定。

让我们看一个简单的异步动作：

```typescript
import { Injectable } from '@angular/core';
import { State, Action, StateContext } from '@ngxs/store';
import { tap } from 'rxjs/operators';

export class FeedAnimals {
  static readonly type = '[Zoo] FeedAnimals';
  constructor(public animalsToFeed: string) {}
}

export interface ZooStateModel {
  feedAnimals: string[];
}

@State<ZooStateModel>({
  name: 'zoo',
  defaults: {
    feedAnimals: []
  }
})
@Injectable()
export class ZooState {
  constructor(private animalService: AnimalService) {}

  @Action(FeedAnimals)
  feedAnimals(ctx: StateContext<ZooStateModel>, action: FeedAnimals) {
    return this.animalService.feed(action.animalsToFeed).pipe(
      tap(animalsToFeedResult => {
        const state = ctx.getState();
        ctx.setState({
          ...state,
          feedAnimals: [...state.feedAnimals, animalsToFeedResult]
        });
      })
    );
  }
}
```

在此示例中，我们联系动物服务并调用`feed`，然后使用`setState`返回结果。 请记住，由于state属性是返回至当前状态切片，因此我们可以保证状态是新的。

您可能会注意到我返回了Observable，只是使用了`tap`。 如果我们返回Observable，框架将自动为我们订阅它，因此我们不必自己处理它。 另外，如果我们希望 stores `dispatch`函数只能在操作完成后才能完成，则需要返回它，以便知道。

Observables are not a requirement, you can use promises too. We could swap that observable chain to look like this:

```typescript
import { Injectable } from '@angular/core';
import { State, Action } from '@ngxs/store';

export class FeedAnimals {
  static readonly type = '[Zoo] FeedAnimals';
  constructor(public animalsToFeed: string) {}
}

export interface ZooStateModel {
  feedAnimals: string[];
}

@State<ZooStateModel>({
  name: 'zoo',
  defaults: {
    feedAnimals: []
  }
})
@Injectable()
export class ZooState {
  constructor(private animalService: AnimalService) {}

  @Action(FeedAnimals)
  async feedAnimals(ctx: StateContext<ZooStateModel>, action: FeedAnimals) {
    const result = await this.animalService.feed(action.animalsToFeed);
    const state = ctx.getState();
    ctx.setState({
      ...state,
      feedAnimals: [...state.feedAnimals, result]
    });
  }
}
```

### 从动作分派动作

如果您想让您的动作分派另一个动作，则可以使用状态上下文对象中包含的 `dispatch` 函数。

```typescript
import { Injectable } from '@angular/core';
import { State, Action, StateContext } from '@ngxs/store';
import { map } from 'rxjs/operators';

export interface ZooStateModel {
  feedAnimals: string[];
}

@State<ZooStateModel>({
  name: 'zoo',
  defaults: {
    feedAnimals: []
  }
})
@Injectable()
export class ZooState {
  constructor(private animalService: AnimalService) {}

  /**
   * Simple Example
   */
  @Action(FeedAnimals)
  feedAnimals(ctx: StateContext<ZooStateModel>, action: FeedAnimals) {
    const state = ctx.getState();
    ctx.setState({
      ...state,
      feedAnimals: [...state.feedAnimals, action.animalsToFeed]
    });

    return ctx.dispatch(new TakeAnimalsOutside());
  }

  /**
   * Async Example
   */
  @Action(FeedAnimals)
  feedAnimals2(ctx: StateContext<ZooStateModel>, action: FeedAnimals) {
    return this.animalService.feed(action.animalsToFeed).pipe(
      tap(animalsToFeedResult => {
        const state = ctx.getState();
        ctx.patchState({
          feedAnimals: [...state.feedAnimals, animalsToFeedResult]
        });
      }),
      mergeMap(() => ctx.dispatch(new TakeAnimalsOutside()))
    );
  }
}
```

注意，我返回了dispatch函数，这返回到上面的示例，其中包含异步操作和分派器订阅结果。 这不是必需的。

