# 设计指南

以下是有关命名和样式约定的建议。

## 状态后缀

状态应始终以`State`一词作为后缀。 首选：`ZooState` 而不是 `Zoo`

## 状态文件名

状态文件名称应带有`.state.ts`后缀

## 状态接口

状态接口应命名为状态名称，后跟 `Model` 后缀。 如果我的状态称为 `ZooState` ，则将其称为状态接口 `ZooStateModel`。

## 选择后缀

选择项应带有 `$` 后缀。 首选：`animals$`，而不是 `animals`

## 插件后缀

插件应以 `Plugin` 后缀结尾

## 插件文件名

插件文件名应以 `.plugin.ts` 结尾

## 文件夹组织

全局状态应放在 `src/shared/state`里面。 功能状态应位于相应的功能文件夹结构 `src/app/my-feature` 中。 动作可以存在于状态文件中，但建议使用单独的文件，例如：`zoo.actions.ts`。

## 动作后缀

动作不应有后缀

## 单元测试

状态的单元测试应命名为 `my-state-name.state.spec.ts` 的样子

## 动作的操作

动作不应处理与视图相关的操作\(如：显示弹出窗口\)。 应使用操作流来处理这类操作

## 避免在状态中保存基于类的实例


状态存储的对象应该是不可变的，并且应该支持序列化和反序列化。 因此，建议在您的状态下存储纯对象文字。 基于类的实例在序列化和反序列化方面并非易事，而且它们常常通过公开操作封装内部结构和改变内部状态。 这与状态中存储的数据要求不符。

这也适用于使用数据集（例如Set，Map，WeakMap，WeakSet等）。由于它们不适合反序列化，因此不易呈现以进行标准化。

### 避免

```typescript
export class Todo {
  constructor(public title: string, public isCompleted = false) {}
}

@State<Todo[]>({
  name: 'todos',
  defaults: []
})
@Injectable()
class TodosState {
  @Action(AddTodo)
  add(ctx: StateContext<Todo[]>, action: AddTodo): void {
    // Avoid new Todo(title)
    ctx.setState((state: Todo[]) => state.concat(new Todo(action.title)));
  }
}

@Component({
  selector: 'app',
  template: `
    <ng-container *ngFor="let todo of todos$ | async">
      {{ todo.isCompleted }}
    </ng-container>
  `
})
class AppComponent {
  @Select(TodosState) todos$: Observable<Todo[]>;
}
```

不建议将基于类的对象实例添加到您的状态，因为这将可能导致未定义的行为。

### 推荐

```typescript
export interface TodoModel {
  title: string;
  isCompleted: boolean;
}

@State<TodoModel[]>({
  name: 'todos',
  defaults: []
})
@Injectable()
class TodosState {
  @Action(AddTodo)
  add(ctx: StateContext<TodoModel[]>, action: AddTodo): void {
    ctx.setState((state: TodoModel[]) =>
      state.concat({ title: action.title, isCompleted: false })
    );
  }
}

@Component({
  selector: 'app',
  template: `
    <ng-container *ngFor="let todo of todos$ | async">
      {{ todo.isCompleted }}
    </ng-container>
  `
})
class AppComponent {
  @Select(TodosState) todos$: Observable<Todo[]>;
}
```

## 展平深层对象图(Flatten Deep Object Graphs)

在Redux中处理分层数据的一般建议是对其进行规范化。 这将需要以与设计关系表相同的方式对其进行展平，它具有引用父对象的键。

### 避免

```typescript
export interface RowStateModel {
  id: number;
}

export interface GridStateModel {
  id: number;
  rows: Map<number, RowState>;
}

export interface GridCollectionStateModel {
  grids: Map<number, GridState>;
}

@State<RowStateModel>({
  name: 'row',
  defaults: {
    id: -1
  }
})
@Injectable()
export class RowState {}

@State<GridStateModel>({
  name: 'grid',
  defaults: {
    id: -1,
    rows: new Map<number, RowState>()
  }
})
@Injectable()
export class GridState {}

@State<GridCollectionStateModel>({
  name: 'grid-collection',
  defaults: {
    grids: new Map<number, GridState>()
  }
})
@Injectable()
export class GridCollectionState {}
```

注意：不建议使用Set，Map，WeakMap，WeakSet等数据收集。因为它们不适合反序列化，并且不易呈现以进行标准化。

### 推荐

```typescript
export interface RowStateModel {
  id: number;
}

export interface GridStateModel {
  id: number;
  rows: {
    [id: number]: RowStateModel;
  };
}

export interface GridCollectionStateModel {
  grids: {
    [id: number]: GridStateModel;
  };
}

@State<RowStateModel>({
  name: 'row',
  defaults: {
    id: -1
  }
})
@Injectable()
export class RowState {}

@State<GridStateModel>({
  name: 'grid',
  defaults: {
    id: -1,
    rows: {}
  }
})
@Injectable()
export class GridState {}

@State<GridCollectionStateModel>({
  name: 'grid-collection',
  defaults: {
    grids: {}
  }
})
@Injectable()
export class GridCollectionState {}
```

