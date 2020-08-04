# Forms

通常，在Angular中构建响应式表单时，您需要将存储中的值绑定到表单，反之亦然。 存储中的值是可观察到的，并且响应形式接受原始对象，结果我们终将来来回回地使用猴子补丁\(monkey patching\)。

除了这些问题之外，您还需要在工作流程中填写表格并离开，然后返回并恢复您的当前状态。 这是存储的绝佳用例，我们可以使用此插件来解决该用例。

简而言之，此插件帮助您让表单和状态保持同步。

## 安装

```bash
npm install @ngxs/form-plugin --save

# or if you are using yarn
yarn add @ngxs/form-plugin
```

## 使用

在应用程序的根模块中导入`NgxsFormPluginModule`，并包含在imports数组中。

```typescript
import { NgxsFormPluginModule } from '@ngxs/form-plugin';
import { NovelsState } from './novels.state';

@NgModule({
  imports: [NgxsModule.forRoot([NovelsState]), NgxsFormPluginModule.forRoot()]
})
export class AppModule {}
```

如果您的表单是在子模块中使用的，则还必须将其导入其中：

```typescript
import { NgxsFormPluginModule } from '@ngxs/form-plugin';

@NgModule({
  imports: [NgxsFormPluginModule]
})
export class SomeModule {}
```

### 表单状态

将默认表单状态定义为应用程序状态的一部分。

```typescript
import { Injectable } from '@angular/core';
import { State } from '@ngxs/store';

@State({
  name: 'novels',
  defaults: {
    newNovelForm: {
      model: undefined,
      dirty: false,
      status: '',
      errors: {}
    }
  }
})
@Injectable()
export class NovelsState {}
```

### 表单设置

在组件中，实现响应形式，并使用含有状态对象的路径的 `ngxsForm` 指令来装饰表单。  我们正将 _string_ 路径传递给 `ngxsForm` 。该指令使用这个路径将自身连接到存储和设置绑定。

```typescript
import { Component } from '@angular/core';
import { FormGroup, FormControl } from '@angular/forms';

@Component({
  selector: 'new-novel-form',
  template: `
    <form [formGroup]="newNovelForm" ngxsForm="novels.newNovelForm" (ngSubmit)="onSubmit()">
      <input type="text" formControlName="novelName" />
      <button type="submit">Create</button>
    </form>
  `
})
export class NewNovelComponent {
  newNovelForm = new FormGroup({
    novelName: new FormControl()
  });

  onSubmit() {
    //
  }
}
```

现在，无论何时您的表单更新，您的状态也将反映新状态。

该指令还可以使用两个值,你可以使用：

* `ngxsFormDebounce: number` - 表单去抖动值，默认为: `100`. 如果`updateOn`是`blur`或`submit`则被忽略。
* `ngxsFormClearOnDestroy: boolean` - 销毁表单时清除状态。

### 动作

除了自动跟踪表单外，您还可以手动调度诸如重置表单状态之类的操作。 例如：

```typescript
this.store.dispatch(
  new UpdateFormDirty({
    dirty: false,
    path: 'novels.newNovelForm'
  })
);
```

表单插件开箱即用，具有以下 `actions` ：

* `UpdateFormStatus({ status, path })` - 更新表单状态
* `UpdateFormValue({ value, path, propertyPath? })` - 更新表单值 \(或可选的内部属性值\)
* `UpdateFormDirty({ dirty, path })` - 更新表单脏状态
* `SetFormDisabled(path)` - 将表单设置为禁用
* `SetFormEnabled(path)` - 将表单设置为启用
* `SetFormDirty(path)` - 将表单设置为脏 \(`UpdateFormDirty`的快捷方式\)
* `SetFormPristine(path)` - 将表单设置为原始 \(`UpdateFormDirty`的快捷方式\)
* `ResetForm({ path, value? })` - 使用(或不使用)表单值重置表单。

### 更新特定的表单属性

表单插件公开了 `UpdateFormValue` 动作，该动作提供了 `propertyPath` 参数，从而可以更新嵌套的表单属性。

```typescript
interface NovelsStateModel {
  newNovelForm: {
    model?: {
      novelName: string;
      authors: {
        name: string;
      }[];
    };
  };
}

@State<NovelsStateModel>({
  name: 'novels',
  defaults: {
    newNovelForm: {
      model: undefined
    }
  }
})
@Injectable()
export class NovelsState {}
```

这个状态包含有关新小说名称及其作者的信息。 让我们创建一个组件，该组件将使用 `ngxsForm` 指令来实现响应形式：

```typescript
@Component({
  selector: 'new-novel-form',
  template: `
    <form [formGroup]="newNovelForm" ngxsForm="novels.newNovelForm" (ngSubmit)="onSubmit()">
      <input type="text" formControlName="novelName" />

      <div
        formArrayName="authors"
        *ngFor="let author of newNovelForm.get('authors').controls; index as index"
      >
        <div [formGroupName]="index">
          <input formControlName="name" />
        </div>
      </div>

      <button type="submit">Create</button>
    </form>
  `
})
export class NewNovelComponent {
  newNovelForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.newNovelForm = this.fb.group({
      novelName: 'Zenith',
      authors: this.fb.array([
        this.fb.group({
          name: 'Sasha Alsberg'
        })
      ])
    });
  }

  onSubmit() {
    //
  }
}
```

让我们再次看一下上面的组件。 假设我们想从应用程序中的任何位置更新表单中第一作者的姓名。 该代码如下所示：

```typescript
store.dispatch(
  new UpdateFormValue({
    path: 'novels.newNovelForm',
    value: {
      name: 'Lindsay Cummings'
    },
    propertyPath: 'authors.0'
  })
);
```

