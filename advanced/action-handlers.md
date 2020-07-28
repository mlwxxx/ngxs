# 动作处理程序

在阅读本文之前-我们建议您开始熟悉[动作的生命周期](actions-life-cycle.md)。

所谓事件源 就是应用程序进行的状态更改所构建的事件的不可变序列或“日志”。 您无需关注当前状态，而可以关注随时间变化的变化。 这是将系统构建为一系列事件的实践。 在NGXS中，我们将其称为“动作处理程序”。

通常，操作直接与状态更改相对应，但是很难使您的组件根据状态一直做出反应。这种范例的副作用，我们最终创建了许多中间状态属性来执行诸如重置表单等操作。 动作处理程序使我们能够基于状态以及发出的事件来驱动组件。

例如，如果我们有购物车并且要从中删除商品，则您可能希望显示一条通知，提示该商品已成功删除。 在纯状态驱动的应用程序中，您可以创建某种消息数组以使对话框显示出来。 使用动作处理程序，我们可以直接响应动作。

动作处理程序是一个可观察对像，它接收在状态对其执行任何操作之前调度的所有动作。

NGXS中的动作也具有生命周期。 由于任何潜在的动作都可能是异步的，因此我们对显示“何时”，“成功”，“取消”或“错误”的动作进行标记。 这使您能够对动作的不同点做出反应。

由于它是可观察的，因此我们可以使用以下管道：

* `ofAction`: 当以下任何生命周期事件发生时触发
* `ofActionDispatched`: 当动作被调度的时候触发
* `ofActionSuccessful`: 当动作成功完成时触发
* `ofActionCanceled`: 当动作被取消的时候触发
* `ofActionErrored`: 当动作导致引发错误时触发
* `ofActionCompleted`: 动作完成时触发，无论它是否成功\(返回完成的摘要信息\)

All of the above pipes return the original `action` in the observable except for the `ofActionCompleted` pipe which returns some summary information for the completed action. This summary is an object with the following interface:
以上所有管道均以可观察到的形式返回原始的 `action` ，但`ofActionCompleted`管道除外，该管道为已完成的操作返回一些摘要信息。 此摘要是具有以下接口的对象：

```typescript
interface ActionCompletion<T = any> {
  action: T;
  result: {
    successful: boolean;
    canceled: boolean;
    error?: Error;
  };
}
```

下面这个动作处理程序，它过滤`RouteNavigate`动作，然后告诉路由器导航到该路由。

```typescript
import { Injectable } from '@angular/core';
import { Actions, ofActionDispatched } from '@ngxs/store';

@Injectable({ providedIn: 'root' })
export class RouteHandler {
  constructor(private router: Router, private actions$: Actions) {
    this.actions$
      .pipe(ofActionDispatched(RouteNavigate))
      .subscribe(({ payload }) => this.router.navigate([payload]));
  }
}
```

记住，您需要确保将 `RouteHandler` 注入到应用程序中的某个位置以供DI连接。 如果您希望它在应用程序启动时发生，Angular提供了一种执行此操作的方法：

```typescript
import { NgModule, APP_INITIALIZER } from '@angular/core';

// Noop handler for factory function
export function noop() {
  return function() {};
}

@NgModule({
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: noop,
      deps: [RouteHandler],
      multi: true
    }
  ]
})
export class AppModule {}
```

动作处理程序也可以在组件中使用。 给定购物车删除示例，我们可以构建类似以下内容的示例：

```typescript
@Component({ ... })
export class CartComponent {
  constructor(private actions$: Actions) {}

  ngOnInit() {
    this.actions$.pipe(ofActionSuccessful(CartDelete)).subscribe(() => alert('Item deleted'));
  }
}
```

使用以下方式取消订阅动作处理程序：

```typescript
@Component({ ... })
export class CartComponent implements OnInit, OnDestroy {
  private ngUnsubscribe = new Subject();

  constructor(private actions$: Actions) {}

  ngOnInit() {
    this.actions$
      .pipe(
        ofActionSuccessful(CartDelete),
        takeUntil(this.ngUnsubscribe)
      )
      .subscribe(() => alert('Item deleted'));
  }

  ngOnDestroy() {
    this.ngUnsubscribe.next();
    this.ngUnsubscribe.complete();
  }
}
```

