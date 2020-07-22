# Select

Selects 函数是从全局状态容器中切片状态(state)的特定部分。


在CQRS和Redux模式中，我们将READ和WRITE分开。NGXS中也存在此模式,当我们想从store中读取数据时，我们使用select运算符来检索该数据。

的NGXS, 有两种选择状态(state)的方法，我们可以在`Store`服务上调用`select`方法，也可以使用@Select装饰器。
首先让我们看一下@Select装饰器。

## 选择装饰器(Select Decorators)

You can select slices of data from the store using the `@Select` decorator. It has a few different ways to get your data out, whether passing the state class, a function, a different state class or a memoized selector.

您可以使用`@Select`装饰器从存储(store)中选择数据片段。 它有几种不同的方式来获取数据，无论是传递状态类，函数，其他状态类还是备注选择器。

```typescript
import { Select } from '@ngxs/store';
import { ZooState, ZooStateModel } from './zoo.state';

@Component({ ... })
export class ZooComponent {
  // Reads the name of the state from the state class
  @Select(ZooState) animals$: Observable<string[]>;

  // Uses the pandas memoized selector to only return pandas
  @Select(ZooState.pandas) pandas$: Observable<string[]>;

  // Also accepts a function like our select method
  @Select(state => state.zoo.animals) animals$: Observable<string[]>;

  // Reads the name of the state from the parameter
  @Select() zoo$: Observable<ZooStateModel>;
}
```

## Store类的select函数

`Store`类还具有`select`函数：

```typescript
import { Store } from '@ngxs/store';

@Component({ ... })
export class ZooComponent {
  animals$: Observable<string[]>;

  constructor(private store: Store) {
    this.animals$ = this.store.select(state => state.zoo.animals);
  }
}
```

当程序无法使用select装饰器静态声明时，它便非常有用。

还有一个`selectOnce`，它将基本上作为自动为您执行`select().pipe(take(1))` 快捷方法。

这在仅希望检查当前状态而不希望继续观看流的路由守卫中非常有用。 它对于单元测试也很有用。

## Selects快照

在store中，有一个`selectSnapshot`功能，可让您提取原始值。 这在需要获取静态值但不能使用Observable的情况下很有用。 一个很好的用例是需要从auth状态获取令牌的拦截器。

```typescript
@Injectable()
export class JWTInterceptor implements HttpInterceptor {
  constructor(private store: Store) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.store.selectSnapshot<string>((state: AppState) => state.auth.token);
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });

    return next.handle(req);
  }
}
```

## 带有记忆功能的选择器(Selectors)

通常，您会在几个不同的地方使用相同的选择器，或者想要将复杂的选择器与组件分离。 NGXS有一个`@ Selector`装饰器，可以帮助我们。 该装饰器将记住该函数的性能，并自动对要处理的状态部分进行切片。

让我们创建一个选择器，该选择器将返回这些动物中熊猫的列表。

```typescript
import { Injectable } from '@angular/core';
import { State, Selector } from '@ngxs/store';

@State<string[]>({
  name: 'animals',
  defaults: []
})
@Injectable()
export class ZooState {
  @Selector()
  static pandas(state: string[]) {
    return state.filter(s => s.indexOf('panda') > -1);
  }
}
```

注意，`state`只是这个`ZooState` 类的本地状态。 现在在我们的组件中，我们只需执行以下操作：

```typescript
@Component({ ... })
export class AppComponent {
  @Select(ZooState.pandas) pandas$: Observable<string[]>;
}
```

而我们的 `pandas $` 将只返回名称包含panda的动物。

### Selector选项

在`NgxsModule.forRoot`选项中使用`selectorOptions`属性可以在全局级别上配置记忆选择器，\(参见 [Options](../advanced/options.md)\)。

也可以在类或方法级别通过`@SelectorOptions`装饰器提供这些选项，以便在该范围内配置选择器的行为。 可以使用以下选项：

#### `suppressErrors`

* `true` 此时选择器内的任何错误，导致选择器返回 `undefined`.
* `false` 导致这些错误在堆栈中传播，从而触发对导致错误的选择器的评估。
* **注意：** \_在NGXS v4中，它的默认值将更改为 `false`

  在NGXS v3.x 中它的默认值是 `true` 。\_

#### `injectContainerState`

* `true` 此时, 在状态类中定义的所有选择器要接收容器类的状态模型作为其第一个参数。这导致了，在对该状态进行任何更改之后，每个选择器都将重新评估。

  **注意：** _这显然是不理想的，, 因此在NGXS v4中此设置默认值将更改为 `false` in NGXS v4。_

* `false` 将防止把容器状态模型作为 选择器方法 \(在状态类中定义的\) 的第一个参数注入，将连接其他选择器作为方法的参数。

* _在 NGXS v3.x 中它的默认值是 `true`._
* 转到 [这里](select.md#joining-selectors) 查看示例来了解这个设置对选择器的影响。

我们建议在全局级别上设置这些选项，除非您要将应用程序从一种行为过渡到另一种行为，在这种行为下您可以使用此装饰器来逐步引入这种过渡。 例如，NGXS v4将对选择器进行更改，这将影响使用联合选择器的方法\(查阅) [这里](select.md#joining-selectors)\).

我们建议对新项目使用以下全局设置，以最大程度地减少v4升级的影响：

```typescript
{
  // These Selector Settings are recommended in preparation for NGXS v4
  // (See above for their effects)
  suppressErrors: false,
  injectContainerState: false
}
```

### 带参数的记忆选择器

选择器可配置为 能够接受参数。

有两种模式可以实现此目的： [Lazy Selectors](select.md#lazy-selectors) or [Dynamic Selectors](select.md#dynamic-selectors)

#### 惰性选择器(Lazy Selectors)

要创建一个惰性选择器，您需要做的就是从选择器中返回一个函数。 选择器返回的函数将被自动存储，并且当选择器的使用者执行该函数时，将在稍后阶段执行该函数内部的逻辑。 请注意，此函数可以接受任意数量的参数\(或者没有参数\)，因为消费者有责任提供它们。

例如，我可以有一个惰性选择器，它将按类型将我们的熊猫过滤。

```typescript
@State<string[]>({
  name: 'animals',
  defaults: []
})
@Injectable()
export class ZooState {
  @Selector()
  static pandas(state: string[]) {
    return (type: string) => {
      return state.filter(s => s.indexOf('panda') > -1).filter(s => s.indexOf(type) > -1);
    };
  }
}
```

您可以使用`store.select` 并使用`rxjs` `map`管道函数执行惰性函数。

```typescript
import { Store } from '@ngxs/store';
import { map } from 'rxjs/operators';

@Component({ ... })
export class ZooComponent {
  babyPandas$: Observable<string[]>;

  constructor(private store: Store) {
    this.babyPandas$ = this.store
      .select(ZooState.pandas)
      .pipe(map(filterFn => filterFn('baby')));
  }
}
```

#### 动态选择器(Dynamic Selectors)

动态选择器是通过使用 `createSelector`函数创建的，而不是`@Selector`装饰器。 无需在任何特定时间在任何特殊区域中创建它。 不过，典型的用例是像普通选择器一样创建，但是需要一个参数来提供给动态选择器。

例如，我可以有一个动态选择器，该选择器可以按类型将我们的熊猫过滤。

```typescript
@State<string[]>({
  name: 'animals',
  defaults: []
})
@Injectable()
export class ZooState {
  static pandas(type: string) {
    return createSelector([ZooState], (state: string[]) => {
      return state.filter(s => s.indexOf('panda') > -1).filter(s => s.indexOf(type) > -1);
    });
  }
}
```

then you can use `@Select` to call this function with the parameter provided.
您可以使用`@Select`来通过提供的参数调用此函数。

```typescript
import { Store } from '@ngxs/store';
import { map } from 'rxjs/operators';

@Component({ ... })
export class ZooComponent {

  @Select(ZooState.pandas('baby'))
  babyPandas$: Observable<string[]>;

  @Select(ZooState.pandas('adult'))
  adultPandas$: Observable<string[]>;

}
```

请注意，这些选择器中的每一个都有各自独立的备注。 即使为以这种方式创建的两个动态选择器提供了相同的参数，它们也将具有独立的备注。

这些选择器非常强大，是在后台创建所有其他选择器的工具。

_动态选择器s \(动态状态切片\)_

一个有趣的用例，是从具有相同结构的状态(States)中进行选择时，允许重用选择器。 例如：

```typescript
export class SharedSelectors {
  static getEntities(stateClass) {
    return createSelector([stateClass], (state: { entities: any[] }) => {
      return state.entities;
    });
  }
}
```

居然可以这样使用：

```typescript
@Component({ ... })
export class ZooComponent {

  @Select(SharedSelectors.getEntities(ZooState))
  zoos$: Observable<Zoo[]>;

  @Select(SharedSelectors.getEntities(ParkState))
  parks$: Observable<Park[]>;

}
```

### 连接选择器(Joining Selectors)

定义选择器时，你还可以将其他选择器传递到`Selector`装饰器的签名中，以将其他选择器与此状态选择器连接在一起。

如果不更改选择器选项 \(查看 [above](select.md#selector-options)\) 那么在NGXS v3.x中，这些选择器将具有以下签名 :

```typescript
@State<PreferencesStateModel>({ ... })
@Injectable()
export class PreferencesState { ... }

@State<string[]>({ ... })
@Injectable()
export class ZooState {

  @Selector([PreferencesState])
  static firstLocalPanda(state: string[], preferencesState: PreferencesStateModel) {
    return state.find(
      s => s.indexOf('panda') > -1 && s.indexOf(preferencesState.location)
    );
  }

  @Selector([ZooState.firstLocalPanda])
  static happyLocalPanda(state: string[], panda: string) {
    return 'happy ' + panda;
  }

}
```

在这里，您可以看到，在状态类中使用带有参数的`Selector`装饰器时，它将注入状态类的状态模型作为第一个参数，然后按在签名中传递其他选择器的顺序注入其他选择器。 这是[injectContainerState`]（select.md＃injectcontainerstate）选项提供的行为，在NGXS v3.x中默认为`true`。

记忆的选择器将在它们的任何输入参数值更\(无论你是否使用它们\)时重新计算。 在上述行为的情况下，状态类的状态模型作为第一个输入参数被注入，选择器将对此模型的任何更改重新计算。 您会注意到，即使不使用`happyLocalPanda`选择器，也具有`state`依赖性。 即使`firstLocalPanda`值可能没有更改，它也会在每次更改`state`时重新计算。 这是不理想的，因此此默认行为在NGXS v4中正在更改。

在NGXS v4及更高版本中，[`injectContainerState`]（select.md＃injectcontainerstate）选择器选项的默认值将更改为`false`，从而选择器的优化程度更高，因为它们没有将状态模型作为第一个注入参数，除非明确要求。 使用此设置，需要按照以下方式定义选择器：


```typescript
@State<PreferencesStateModel>({ ... })
@Injectable()
export class PreferencesState { ... }

@State<string[]>({ ... })
@Injectable()
export class ZooState {

 @Selector([ZooState, PreferencesState])
 static firstLocalPanda(state: string[], preferencesState: PreferencesStateModel) {
   return state.find(
     s => s.indexOf('panda') > -1 && s.indexOf(preferencesState.location)
   );
 }

 @Selector([ZooState.firstLocalPanda])
 static happyLocalPanda(panda: string) {
   return 'happy ' + panda;
 }

}
```

现在，`happyLocalPanda`只会在`firstLocalPanda`选择器的输出值更改时重新计算。

我们建议您将项目移至此行为，以优化选择器并为NGXS v4中的默认值更改做准备。 请参见 选择器选项 [above](select.md#selector-options)部分了解建议的设置。

### 元选择器(Meta Selectors)

默认情况下，NGXS中的选择器绑定到一个状态。 有时，您需要以高性能可重用的方式链接不相关状态的能力。 元选择器是一个可让您将N个选择器绑定在一起以返回状态流的 选择器。

假设我们有2个状态； '动物园'和'主题公园'('zoos' and 'theme parks')。 我们有一个组件，需要显示某个城市的所有动物园和主题公园。 这是两个非常不同的状态类，它们可能根本不相关。 我们可以使用元选择器将这两个状态结合在一起，例如：

```typescript
export class CityService {
  @Selector([Zoo, ThemePark])
  static zooThemeParks(zoos, themeParks) {
    return [...zoos, ...themeParks];
  }
}
```

现在我们可以在应用程序中的任何地方使用此`zooThemeParks`选择器。

### 交互选择器的顺序(The Order of Interacting Selectors)

在3.6.1之前的NGXS版本中，存在一个问题，即选择器的声明顺序很重要。 这已在PR [\＃1514]（https://github.com/ngxs/store/pull/1514）中修复，现在可以以任意顺序声明选择器。

### 继承选择器(Inheriting Selectors)

当我们拥有共享相似结构的状态时，我们可以将共享的选择器提取到一个基类中，以便以后进行扩展。 如果我们在多个状态上都有一个 `entities` 字段，则可以创建一个包含动态 `@Selector()` 字段的基类，并扩展这样的`@State`类上对其进行,像如下进行。


```typescript
export class EntitiesState {
  static entities<T>() {
    return createSelector([this], (state: { entities: T[] }) => {
      return state.entities;
    });
  }

  //...
}
```

并在每个`@State`上扩展 `EntitiesState` 类，如下所示：

```typescript
export interface UsersStateModel {
  entities: User[];
}

@State<UsersStateModel>({
  name: 'users',
  defaults: {
    entities: []
  }
})
@Injectable()
export class UsersState extends EntitiesState {
  //...
}

export interface ProductsStateModel {
  entities: Product[];
}

@State<ProductsStateModel>({
  name: 'products',
  defaults: {
    entities: []
  }
})
@Injectable()
export class ProductsState extends EntitiesState {
  //...
}
```

然后，您就可以像下面这个使用它们：

```typescript
@Component({ ... })
export class AppComponent {

  @Select(UsersState.entities<User>())
  users$: Observable<User[]>;

  @Select(ProductsState.entities<Product>())
  products$: Observable<Product[]>;

}
```

Or:

```typescript
this.store.select(UsersState.entities<User>());
```

## 特别注意事项

### 角库：在静态函数中使用lambda


_如果要直接构建Angular库，以便可以将其部署到Angular编译器选项`strictMetadataEmit` \(查阅_ [_docs_](https://angular.io/guide/aot-compiler#strictmetadataemit)_\)将最有可能被设置为true，Angular的 `@angular/compiler-cli` 包中的 `MetadataCollector` 将报告在静态方法中使用lambda的以下问题:_

> Metadata collected contains an error that will be reported at runtime: Lambda not supported.\`
> 收集的元数据包含将在运行时报告的错误：不支持Lambda。\`

对于下面定义的每个选择器，都会报告此错误，但是，如示例中所示，您可以通过在类表达式和修饰符之前包含 `// @ dynamic` 注释来防止此错误：

```typescript
// @dynamic
@State<string[]>({
  name: 'animals',
  defaults: ['panda', 'horse', 'bee']
})
@Injectable()
export class ZooState {
  @Selector()
  static pandas(state: string[]) {
    return state.filter(s => s.indexOf('panda') > -1);
  }

  @Selector()
  static horses(state: string[]) {
    return (type: string) => {
      return state.filter(s => s.indexOf('horse') > -1).filter(s => s.indexOf(type) > -1);
    };
  }

  static bees(type: string) {
    return createSelector([ZooState], (state: string[]) => {
      return state.filter(s => s.indexOf('bee') > -1).filter(s => s.indexOf(type) > -1);
    });
  }
}
```

或者，您可以在返回结果之前将结果分配给变量：
查阅 [https://github.com/ng-packagr/ng-packagr/issues/696\#issuecomment-387114613](https://github.com/ng-packagr/ng-packagr/issues/696#issuecomment-387114613)

```typescript
@State<string[]>({
  name: 'animals',
  defaults: ['panda', 'horse', 'bee']
})
@Injectable()
export class ZooState {
  @Selector()
  static pandas(state: string[]) {
    const result = state.filter(s => s.indexOf('panda') > -1);
    return result;
  }

  @Selector()
  static horses(state: string[]) {
    const fn = (type: string) => {
      return state.filter(s => s.indexOf('horse') > -1).filter(s => s.indexOf(type) > -1);
    };
    return fn;
  }

  static bees(type: string) {
    const selector = createSelector([ZooState], (state: string[]) => {
      return state.filter(s => s.indexOf('bee') > -1).filter(s => s.indexOf(type) > -1);
    });
    return selector;
  }
}
```

### 使用选择装饰器 with `strictPropertyInitialization`

如果启用了 `strictPropertyInitialization` 选项，那么TypeScript编译器将要求在构造函数中显式初始化所有类属性。 给出以下代码：

```typescript
@Component({ ... })
export class ZooComponent {
  @Select(ZooState.pandas) pandas$: Observable<string[]>;
}
```

在上面的示例中，仅当打开 `strictPropertyInitialization` 时，编译器才会发出以下错误：

```text
// Type error: Property 'pandas$' has no initializer
// and is not definitely assigned in the constructor
```

我们可以通过应用 [definite assignment assertion](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-7.html#definite-assignment-assertions) 对 `pandas$` 属性声明 \(注意后面的感叹号\):

```typescript
@Component({ ... })
export class ZooComponent {
  @Select(ZooState.pandas) pandas$!: Observable<string[]>;
}
```

通过添加确定的赋值断言，我们告诉类型检查器我们确定 `pandas$` 属性将被初始化（通过 `@Select` 装饰器）。

