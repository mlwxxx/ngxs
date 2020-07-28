# 组合

您可以使用类继承将多个存储组合在一起。 这很简单：

```typescript
@State({
  name: 'zoo',
  defaults: {
    type: null
  }
})
@Injectable()
class ZooState {
  @Action(Eat)
  eat(ctx: StateContext) {
    ctx.setState({ type: 'eat' });
  }
}

@State({
  name: 'stlzoo'
})
@Injectable()
class StLouisZooState extends ZooState {
  @Action(Drink)
  drink(ctx: StateContext) {
    ctx.setState({ type: 'drink' });
  }
}
```

这样，当调用 `StLouisZooState` 时，它将共享 `ZooState` 的动作。 同样，所有状态选项都被继承。

