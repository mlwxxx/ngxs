# NGXS的组件事件

Developers always use the `@Output` decorator in conjunction with the `EventEmitter`. The below code has been seen by any Angular developer:
开发人员通常将 `@Output` 装饰器与 `EventEmitter` 结合使用。 任何Angular开发人员都可以看到以下代码：

```typescript
@Output() search = new EventEmitter<string>();
```

秘密是`@Output`可以修饰任何“可观察”的属性。 Angular编译器为Angular本身发出了必要的信息，说：“嘿，请订阅 `search` 类属性，并在可观察对象发出时随时调度 `CustomEvent` 。

假设我们是A团队的一员。 我们开发了使用NGXS的自定义元素，并且希望将此组件提供给团队B。团队B对NGXS一无所知，他们不能使用我们的API。 我们的元素只是一个黑盒，它只通过`@Output`公开数据。

我们开发了一个 `app-email-list` 自定义元素，该元素会发出`messagesLoaded`DOM事件，并将数据提供给团队B进行分析。 给出以下代码：

```typescript
@Component({
  selector: 'app-email-list',
  template: `
    <app-message *ngFor="let message of messages$ | async" [message]="message"></app-message>
    <app-button (click)="refresh()">Refresh messages</app-button>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class EmailListComponent {
  @Select(MessagesState.getMessages) messages$: Observable<Message[]>;

  @Output() messagesLoaded = new EventEmitter<Message[]>();

  constructor(private store: Store) {}

  refresh(): void {
    this.store.dispatch(new LoadMessages()).subscribe(() => {
      const messages = this.store.selectSnapshot(MessagesState.getMessages);
      this.messagesLoaded.emit(messages);
    });
  }
}
```

上面的代码非常简单，仅用于演示！ 如您所见，每当用户单击“刷新消息”按钮时，我们都会调度 `LoadMessages` 操作。 在 `LoadMessages` 动作处理程序完成其异步工作之后，我们发出 `messagesLoaded` 事件。 让我们更具声明性：

```typescript
@Component({
  selector: 'app-email-list',
  template: `
    <app-message *ngFor="let message of messages$ | async" [message]="message"></app-message>
    <app-button>Refresh messages</app-button>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class EmailListComponent {
  @Select(MessagesState.getMessages) messages$: Observable<Message[]>;

  @ViewChild(ButtonComponent, { static: true }) button: ButtonComponent;

  @Output() messagesLoaded = this.button.click.pipe(
    switchMap(() => this.store.dispatch(new LoadMessages())),
    map(() => this.store.selectSnapshot(MessagesState.getMessages))
  );

  constructor(private store: Store) {}
}
```


假设 `ButtonComponent.click` 是一个 `EventEmitter`。我们以更具声明性和反应性的方式做到了。 因此，当用户单击 `app-button` 时，我们的 `switchMap` 将产生下一个 `store.dispatch` 订阅，并从上一个中取消。接下来，我们使用 `map` 运算符，它会将流的值从状态映射到`Message[]`数组。

现在，让我们与A和B团队一起消除这个想法。 由于我们的存储(store)是真相的唯一来源，因此我们可以侦听应用程序任何部分的任何操作。 DOM事件可以很方便地与`Actions`流一起使用。 假设我们有一个组件，每次加载不同类型的书籍时都会发出 `booksLoaded` 事件：


```typescript
// books.state.ts
const enum Genre {
  Novel,
  Detective,
  Horror
}

export class LoadBooks {
  static readonly type = '[Books] Load books';
  constructor(public genre: Genre) {}
}

export class BooksState {
  static getBooks(genre: Genre) {
    return createSelector(
      [BooksState],
      (books: Book[]) => books.filter(book => book.genre === genre)
    );
  }
}

// books.component.ts
export class BooksComponent {
  @Output() booksLoaded = this.actions$.pipe(
    ofActionSuccessful(LoadBooks),
    map((action: LoadBooks) => this.store.selectSnapshot(BooksState.getBooks(action.genre)))
  );

  constructor(private store: Store, private actions$: Actions) {}
}
```

这可能会大大减少您的代码业务逻辑，并以更具声明性和反应性的方式来实现。

