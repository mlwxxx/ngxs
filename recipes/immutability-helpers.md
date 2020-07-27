# 不可变数据助手

Redux是一个微型的模式，将状态表示为不可变的对象。 Redux最初是为React设计的。 大多数Redux概念（例如纯函数）都围绕React生态系统。 如今，Redux与React没有直接关系。


Redux的基石是不变性。 不变性是一种惊人的模式，可以最大程度地减少代码中不可预测的行为。 本文将不讨论函数式编程。 但是，我们将研究非常有用的称为"不可变数据助手"(immutability helpers)的软件包。

## 问题

大多数开发人员必须处理所谓的“深层对象”，最重要的是要遵循不变性概念，以更改某些深层嵌套属性的值。 给出以下代码：

```typescript
export interface Task {
  title: string;
  dates: {
    startDate: string;
    dueDate: string;
  };
}

export interface TrelloStateModel {
  tasks: {
    [taskId: string]: Task;
  };
}
@State<TrelloStateModel>({
  name: 'trello',
  defaults: {
    tasks: {}
  }
})
@Injectable()
export class TrelloState {}
```


让我们想象一下，我们有一个需要更改 `dueDate` 属性的需求：

```typescript
export class UpdateDueDate {
  static readonly type = '[Trello] Update due date';
  constructor(public taskId: string, public dueDate: string) {}
}
```

让我们看看如何实现 `updateDueDate` 动作处理程序：

```typescript
export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    ctx.setState(state => ({
      tasks: {
        ...state.tasks,
        [action.taskId]: {
          ...state.tasks[action.taskId],
          dates: {
            ...state.tasks[action.taskId].dates,
            dueDate: action.dueDate
          }
        }
      }
    }));
  }
}
```

该代码可以工作，但不幸的是，维护和理解起来很复杂。 它不是自我描述的，对于新开发人员来说将是艰巨的。

## 解决方案

有多种方法可以改进此代码。 让我们看一些可以在这方面有所帮助的软件包。

### 状态操作符(State Operators)

[状态操作符](../advanced/operators.md)是NGXS提供的的一流的不可变数据助手,它开箱即用。如果选择状态运算符作为您的不可变数据助手，那么 `patch` 运算符将成为您最好的朋友。 让我们看看如何在`patch` 状态运算符的帮助下重新编写以上代码：


```typescript
import { patch } from '@ngxs/store/operators';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    ctx.setState(
      patch({
        tasks: patch({
          [action.taskId]: patch({
            dates: patch({
              dueDate: action.dueDate
            })
          })
        })
      })
    );
  }
}
```

### immer

`immer` 是一个非常流行的库，它使您可以对不可变对象进行更改，就好像它们是可变的一样。 以下代码显示了如何在Immer的帮助下编写相同的代码：

```typescript
import { produce } from 'immer';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    const state = produce(ctx.getState(), draft => {
      draft.tasks[action.taskId].dates.dueDate = action.dueDate;
    });

    ctx.setState(state);
  }
}
```

Immer的 `produce` 函数也可以用作状态运算符：

```typescript
import { produce } from 'immer';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    ctx.setState(
      produce(draft => {
        draft.tasks[action.taskId].dates.dueDate = action.dueDate;
      })
    );
  }
}
```

在`immer`库中，您可能关注这会减少多少代码，看起来有多好 ：

> 使用Immer就像有个私人助理； 他拿了一封信\(当前状态\)，并给了您一份副本\(草稿\) 来记录更改。 完成后，助手将使用你的草稿生成真正的不可变对像，即最后的封信 \(下一个状态\)。

[Immer 库](https://github.com/immerjs/immer)

### immutability-helper

`immutability-helper` 是一个很小的包，可让您更改数据副本而无需更改原始源：

```typescript
import update from 'immutability-helper';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    const state = update(ctx.getState(), {
      tasks: {
        [action.taskId]: {
          dates: {
            dueDate: {
              $set: action.dueDate
            }
          }
        }
      }
    });

    ctx.setState(state);
  }
}
```

[immutability-helper 库](https://github.com/kolodny/immutability-helper)

### object-path-immutable

`object-path-immutable` 是一个小型库，可让您修改深层对象属性而不用修改原始对象。 让我们看看如何使用该库编写相同的代码：

```typescript
import immutable from 'object-path-immutable';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    const state = immutable.set(
      ctx.getState(),
      `tasks.${action.taskId}.dates.dueDate`,
      action.dueDate
    );

    ctx.setState(state);
  }
}
```

[object-path-immutable 库](https://github.com/mariocasciaro/object-path-immutable)

### immutable-assign

`immutable-assign` 是追求相同目标的轻量级库。 它的语法类似于 `immer`：

```typescript
import * as iassign from 'immutable-assign';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    const state = iassign(ctx.getState(), state => {
      state.tasks[action.taskId].dates.dueDate = action.dueDate;
      return state;
    });

    ctx.setState(state);
  }
}
```

[immutable-assign 库](https://github.com/engineforce/ImmutableAssign)

### Ramda

Ramda 是用于函数式编程的出色库，并且已在许多项目中使用。 对于在项目中同时使用Ramda和NGXS的人，此示例可能很有用：
```typescript
import * as R from 'ramda';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    const property = R.lensPath(['tasks', action.taskId, 'dates', 'dueDate']);
    const state = R.set(property, action.dueDate, ctx.getState());
    ctx.setState(state);
  }
}
```

[Ramda 库](https://github.com/ramda/ramda)

### icepick

`icepick` 是用于不可变集合的零依赖库。 以下是重写后的代码：

```typescript
import * as icepick from 'icepick';

export class TrelloState {
  @Action(UpdateDueDate)
  updateDueDate(ctx: StateContext<TrelloStateModel>, action: UpdateDueDate) {
    const state = icepick.setIn(
      ctx.getState(),
      ['tasks', action.taskId, 'dates', 'dueDate'],
      action.dueDate
    );

    ctx.setState(state);
  }
}
```

[icepick 库](https://github.com/aearly/icepick)

## 总结

我们研究了几种不同的库，它们可能有助于伴随不可变数据的概念。 你只需要选择一个适合您需求的。

