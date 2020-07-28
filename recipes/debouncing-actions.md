# 防抖动动作

在某些情况下，需要对已调度的动作进行去抖动并减少发送到我们的API的请求。 让我们考虑一个简单的应用程序，该应用程序呈现新闻列表并提供在所有新闻中进行搜索的功能：

```typescript
class SearchNews {
  static readonly type = '[News] Search news';
  constructor(public title: string) {}
}

@Component({
  selector: 'app-news-portal',
  template: `
    <app-news-search
      (search)="search($event)"
      [lastSearchedTitle]="lastSearchedTitle$ | async"
    ></app-news-search>

    <app-news [news]="news$ | async"></app-news>
  `
})
export class NewsPortalComponent implements OnDestroy {
  @Select(NewsState.getNews) news$: Observable<News[]>;

  lastSearchedTitle$ = this.store.selectOnce(NewsState.getLastSearchedTitle);

  private destroy$ = new Subject<void>();

  constructor(private store: Store, actions$: Actions) {
    actions$
      .pipe(
        ofActionDispatched(SearchNews),
        map((action: SearchNews) => action.title),
        debounceTime(2000),
        takeUntil(this.destroy$)
      )
      .subscribe(title => {
        store.dispatch(new GetNews(title));
      });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  search(title: string): void {
    this.store.dispatch(new SearchNews(title));
  }
}
```

在上面的示例中，我们有一个名为 `app-news-portal` 的组件，该组件侦听由`app-news-search` 组件分发 `search` 事件 。 在`search`方法被`search`事件调用，并调度`SearchNews` 动作。 注意，`SearchNews` 动作是在组件文件中定义的，因为它从未被应用程序的任何其他部分使用。 我们不想让服务器请求超载，因此我们会侦听 `Actions` 流，该流通过 `debounceTime` 运算符将 `SearchNews` 操作传递给了管道。 让我们看一下下面的代码，了解如何实现 `NewsState`：

```typescript
export interface NewsStateModel {
  news: News[];
  lastSearchedTitle: string | null;
}

export class GetNews {
  static readonly type = '[News] Get news';
  constructor(public title = '') {}
}

@State<NewsStateModel>({
  name: 'news',
  defaults: {
    news: [],
    lastSearchedTitle: null
  }
})
@Injectable()
export class NewsState {
  @Selector()
  static getNews(state: NewsStateModel): News[] {
    return state.news;
  }

  @Selector()
  static getLastSearchedTitle(state: NewsStateModel): string | null {
    return state.lastSearchedTitle;
  }

  constructor(private http: HttpClient) {}

  @Action(GetNews)
  getNews(ctx: StateContext<NewsStateModel>, { title }: GetNews) {
    return this.http.get<News[]>(`/api/news?search=${title}`).pipe(
      tap(news => {
        ctx.setState({ news, lastSearchedTitle: title });
      })
    );
  }
}
```

上面的状态很简单。 如您所见，我们没有为 `SearchNews` 创建动作处理程序，但仍将通过 `Actions` 流将其传递并进行去抖动。 这完全取决于实际的任务，但是您已经了解去抖动作的相关信息。

