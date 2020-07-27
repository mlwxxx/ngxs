# 单元测试

使用NGXS时可以轻松进行单元测试。 为了执行单元测试，我们只是调度事件，侦听更改并执行想要进行的操作。 基本测试如下所示：

```typescript
import { TestBed } from '@angular/core/testing';

describe('Zoo', () => {
  let store: Store;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [NgxsModule.forRoot([ZooState])]
    });

    store = TestBed.inject(Store);
  });

  it('it toggles feed', () => {
    store.dispatch(new FeedAnimals());

    const feed = store.selectSnapshot(state => state.zoo.feed);
    expect(feed).toBe(true);
  });
});
```

我们建议使用 `selectSnapshot` 方法而不是 `selectOnce` 或 `select` 。 茉莉花（Jasmine）和杰斯特（Jest）可能不会在“订阅”（Subscribe）块内运行。 给出以下示例：

```typescript
it('should select zoo', () => {
  store
    .selectOnce(state => state.zoo)
    .subscribe(zoo => {
      // Note: this expectation will not be run!
      expect(zoo).toBeTruthy();
    });

  const zoo = store.selectSnapshot(state => state.zoo);
  expect(zoo).toBeTruthy();
});
```

## 预备状态

Often times in your app you want to test what happens when the state is C and you dispatch action X. You can use the `store.reset(MyNewState)` to prepare the state for your next operation.
在您的应用中，您通常想测试状态为C并调度动作X时会发生什么。您可以使用 `store.reset(MyNewState)` 来准备用于下一个操作的状态。

注意：如果您重置状态，则需要提供注册状态名称作为密钥。 `store.reset` 将体现您的整个状态！ 用新更改的合并到当前状态，以确保不会丢失任何内容。

```typescript
// zoo.state.spec.ts
import { TestBed } from '@angular/core/testing';

export const SOME_DESIRED_STATE = {
  animals: ['Panda']
};

describe('Zoo', () => {
  let store: Store;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [NgxsModule.forRoot([ZooState])]
    });

    store = TestBed.inject(Store);
    store.reset({
      ...store.snapshot(),
      zoo: SOME_DESIRED_STATE
    });
  });

  it('it toggles feed', () => {
    store.dispatch(new FeedAnimals());

    const feed = store.selectSnapshot(state => state.zoo.feed);
    expect(feed).toBe(true);
  });
});
```

## 测试选择器

选择器只是接受状态作为参数的简单函数，因此测试它们非常容易。 一个简单的测试可能看起来像这样：

```typescript
import { TestBed } from '@angular/core/testing';

describe('Zoo', () => {
  it('it should select pandas', () => {
    const pandas = store.selectSnapshot(Zoo.pandas);
    expect(pandas).toEqual(['pandas']);
  });
});
```

在您的应用程序中，您可能会使用 `createSelector` 函数动态创建选择器：

```typescript
export class ZooSelectors {
  static animalNames = (type: string) => {
    return createSelector([ZooState], (state: ZooStateModel) =>
      state.animals.filter(animal => animal.type === type).map(animal => animal.name)
    );
  };
}
```

测试这些选择器确实很容易。 您只需要模拟状态并将其作为参数传递给我们的选择器：

```typescript
it('should select requested animal names from state', () => {
  const zooState = {
    animals: [
      { type: 'zebra', name: 'Andy' },
      { type: 'panda', name: 'Betty' },
      { type: 'zebra', name: 'Crystal' },
      { type: 'panda', name: 'Donny' }
    ]
  };

  const value = ZooSelectors.animalNames('zebra')(zooState);

  expect(value).toEqual(['Andy', 'Crystal']);
});
```

## 测试异步动作

使用Jasmine或Jest测试异步操作也非常容易。 这些测试框架的最大特点是对 `async/await` 的支持。 没有人会阻止您使用`async/await` +  RxJS `toPromise`方法将 `Observable` 转换为 `Promise` 。 另外，您也可以执行 `done` 回调，Jasmine或Jest将等到调用 `done` 回调后再完成测试。

下面的例子并不是很复杂，但是清楚地显示了如何使用 `async/await` 测试异步代码：

```typescript
import { timer } from 'rxjs';
import { tap, mergeMap } from 'rxjs/operators';

it('should wait for completion of the asynchronous action', async () => {
  class IncrementAsync {
    static type = '[Counter] Increment async';
  }

  class DecrementAsync {
    static type = '[Counter] Decrement async';
  }

  // 假设您将对您的API或其他什么进行XHR调用
  function getRandomDelay() {
    return 1000 * Math.random();
  }

  @State({
    name: 'counter',
    defaults: 0
  })
  @Injectable()
  class CounterState {
    @Action(IncrementAsync)
    incrementAsync(ctx: StateContext<number>) {
      const delay = getRandomDelay();

      return timer(delay).pipe(
        tap(() => {
          // We're incrementing the state value and setting it
          ctx.setState(state => (state += 1));
        }),
        // After incrementing we want to decrement it again to the zero value
        mergeMap(() => ctx.dispatch(new DecrementAsync()))
      );
    }

    @Action(DecrementAsync)
    decrementAsync(ctx: StateContext<number>) {
      const delay = getRandomDelay();

      return timer(delay).pipe(
        tap(() => {
          ctx.setState(state => (state -= 1));
        })
      );
    }
  }

  TestBed.configureTestingModule({
    imports: [NgxsModule.forRoot([CounterState])]
  });

  const store: Store = TestBed.inject(Store);

  await store.dispatch(new IncrementAsync()).toPromise();

  const counter = store.selectSnapshot(CounterState);
  expect(counter).toBe(0);
});
```

