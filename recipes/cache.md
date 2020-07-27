# 缓存

使用Actions执行缓存操作是一种常见的做法。 NGXS并没有提供此功能，但是很容易实现。

有许多不同的方法可以解决此问题。 这个简单示例中，使用存储的当前值并返回它们而不是调用HTTP服务。

```typescript
import { Injectable } from '@angular/core';
import { State, Action, StateContext } from '@ngxs/store';
import { tap } from 'rxjs/operators';

export class GetNovels {
  static readonly type = '[Novels] Get novels';
}

@State<Novel[]>({
  name: 'novels',
  defaults: []
})
@Injectable()
export class NovelsState {
  constructor(private novelsService: NovelsService) {}

  @Action(GetNovels)
  getNovels(ctx: StateContext<Novel[]>) {
    return this.novelsService.getNovels().pipe(tap(novels => ctx.setState(novels)));
  }
}
```

想象一下，这种小说(novels)状态仅包含有关它们的最少信息，例如ID和名称。 当用户选择特定小说时，他将被重定向到包含有关该小说的完整信息的页面。 我们只希望加载一次此信息。 让我们创建一个状态并将其称为`novelsInfo`，这将是对象，其键是小说(novels)的标识符：

```typescript
import { Injectable } from '@angular/core';
import { State, Action, StateContext, createSelector } from '@ngxs/store';
import { tap } from 'rxjs/operators';

export interface NovelsInfoStateModel {
}

export class GetNovelById {
  static readonly type = '[Novels info] Get novel by ID';
  constructor(public id: string) {}
}

@State<NovelsInfoStateModel>({
  name: 'novelsInfo',
  defaults: {}
})
@Injectable()
export class NovelsInfoState {
  static getNovelById(id: string) {
    return createSelector([NovelsInfoState], (state: NovelsInfoStateModel) => state[id]);
  }

  constructor(private novelsService: NovelsService) {}

  @Action(GetNovelById)
  getNovelById(ctx: StateContext<NovelsInfoStateModel>, action: GetNovelById) {
    const novels = ctx.getState();
    const id = action.id;

    if (novels[id]) {
      // If the novel with ID has been already loaded
      // we just break the execution
      return;
    }

    return this.novelsService.getNovelById(id).pipe(
      tap(novel => {
        ctx.patchState({ [id]: novel });
      })
    );
  }
}
```

在组件如果想要显示有关小说的信息，可以订阅`ActivatedRoute`中的`params`可观测对像来侦听到参数变化。 该代码将如下所示：

```typescript
@Component({
  selector: 'app-novel',
  template: `
    <h1>{{ novel?.title }}</h1>
    <span>{{ novel?.author }}</span>
    <p>
      {{ novel?.content }}
      <del datetime="{{ novel?.publishedAt }}"></del>
    </p>
  `
})
export class NovelComponent implements OnDestroy {
  novel: Novel;

  private destroy$ = new Subject<void>();

  constructor(route: ActivatedRoute, store: Store) {
    route.params
      .pipe(
        switchMap(params =>
          store
            .dispatch(new GetNovelById(params.id))
            .pipe(mergeMap(() => store.select(NovelsInfoState.getNovelById(params.id))))
        ),
        takeUntil(this.destroy$)
      )
      .subscribe(novel => {
        this.novel = novel;
      });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

在此示例中，我们使用的是`switchMap`，因此，如果用户导航到另一本小说，并且`params`对像发出新值 - 我们必须完成先前启动的异步作业 \(在本例中，是通过ID获取小说的\)。

