# Authentication

身份验证是许多应用程序中的通用主题。 让我们看一下如何在NGXS中实现这一点。

首先，让我们定义状态模型和动作：

```typescript
export interface AuthStateModel {
  token: string | null;
  username: string | null;
}

export class Login {
  static readonly type = '[Auth] Login';
  constructor(public payload: { username: string; password: string }) {}
}

export class Logout {
  static readonly type = '[Auth] Logout';
}
```


在状态模型中，我们要跟踪令牌和用户名。 该令牌表示为该会话发行的JWT令牌。

让我们在状态类中连接这些操作，并将其连接到我们的登录服务。

```typescript
@State<AuthStateModel>({
  name: 'auth',
  defaults: {
    token: null,
    username: null
  }
})
@Injectable()
export class AuthState {
  @Selector()
  static token(state: AuthStateModel): string | null {
    return state.token;
  }

  @Selector()
  static isAuthenticated(state: AuthStateModel): boolean {
    return !!state.token;
  }

  constructor(private authService: AuthService) {}

  @Action(Login)
  login(ctx: StateContext<AuthStateModel>, action: Login) {
    return this.authService.login(action.payload).pipe(
      tap((result: { token: string }) => {
        ctx.patchState({
          token: result.token,
          username: action.payload.username
        });
      })
    );
  }

  @Action(Logout)
  logout(ctx: StateContext<AuthStateModel>) {
    const state = ctx.getState();
    return this.authService.logout(state.token).pipe(
      tap(() => {
        ctx.setState({
          token: null,
          username: null
        });
      })
    );
  }
}
```


在这个状态类中，我们有：

* 一个选择器，它将从储存(store)中选择令牌
* 登录操作方法，它将调用身份验证服务并设置令牌
* 注销操作方法，它将调用身份验证服务并删除我们的状态

现在，在我们的模块中连接状态。

```typescript
@NgModule({
  imports: [
    NgxsModule.forRoot([AuthState]),
    NgxsStoragePluginModule.forRoot({
      key: 'auth.token'
    })
  ]
})
export class AppModule {}
```

In a typical JWT setup, you want to store your token in the `localstorage`. To do this so we hookup our storage plugin and tell it to track the token key in our state.
在典型的JWT设置中，您要将令牌存储在`localstorage`中。 为此，我们连接了存储插件，并告诉它在我们的状态中跟踪令牌密钥。

接下来，我们要确保我们的用户不能进入任何需要身份验证的页面。 我们可以使用Angular提供的路由器防护功能轻松完成此任务。

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private store: Store) {}

  canActivate() {
    const isAuthenticated = this.store.selectSnapshot(AuthState.isAuthenticated);
    return isAuthenticated;
  }
}
```

我们通过路由卫士决定某个路由是否能用我们从store中选择的令牌来激活。

如果令牌无效，则不会让用户转到该页面。 我们确保在路由 `canActivate` 中定义 `AuthGuard` 来实现它。

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: './admin/admin.module#AdminModule',
    canActivate: [AuthGuard]
  }
];
```

您要执行的常见操作是，当用户注销时，我们实际上希望将用户重定向到登录页面。 我们可以使用我们的动作流来侦听 `Logout` 动作，并告诉路由器进入登录页面。

```typescript
@Component({
  selector: 'app',
  template: '..'
})
export class AppComponent implements OnInit {
  constructor(private actions: Actions, private router: Router) {}

  ngOnInit() {
    this.actions.pipe(ofActionDispatched(Logout)).subscribe(() => {
      this.router.navigate(['/login']);
    });
  }
}
```

就是这样！

