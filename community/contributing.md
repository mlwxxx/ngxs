# 贡献

我们希望您能为我们的项目做出贡献，并帮助使其变得更好！ 作为贡献者，以下是我们希望您遵循的准则。

* [报告问题](contributing.md#report-issues)
* [申请功能](contributing.md#request-features)
* [提交Pull Request \(PR\)](contributing.md#submitting-a-pull-request-pr)
* [提交消息格式](contributing.md#commit-message-format)

## 报告问题

如果您在源代码中发现错误或在文档中发现错误，则可以通过将问题提交到我们的GitHub来帮助我们。 包括问题重现\(通过Stackblitz，Plunkr等\) 是帮助团队快速诊断问题的绝对最佳方法。 屏幕截图也很有帮助。

您可以提供更多帮助，并提交包含修复程序的请求请求。

## 提交礼仪

在您提交问题\(issue\)之前，请搜索档案，也许您的问题已被回答。

如果您的问题似乎是Bug，并且尚未报告，请打开一个新问题。 不报告重复问题，帮助我们最大程度地投入精力来解决问题和添加新功能。 提供以下信息将增加迅速解决您的问题的机会：

* 问题\(Issue\) 概述 - 如果抛出错误，则非最小化堆栈跟踪会有所帮助。
* Angular和ngxs版本 - 哪些版本的Angular和ngxs会受到影响
* 动机或用例 - 解释您要做什么以及为什么当前行为对您来说是个错误
* 浏览器和操作系统 - 是不是所有浏览器都有此问题。
* 重现错误 - 提供实时示例\(使用Stackblitz，Plunker等\)或明确的步骤集
* 屏幕截图 - 由于ngxs的视觉特性，屏幕截图可以比文本描述更快地帮助团队分类问题。
* 相关问题 - 以前有没有报道过类似的问题？
* 修复建议 - 如果您自己无法修复该错误，也许您可以指出可能导致该问题的原因\(代码行或提交行\)

## 申请功能

您可以通过向我们的GitHub提交issue来请求新功能。 如果您想实施一项新功能，请先提交一份有关您的工作的建议书，以确保我们可以采纳它。 请考虑这是什么变化：

* 对于主要功能，请先打开一个issue并概述您的建议，以便进行讨论。

  这也将使我们能够更好地协调我们的工作，防止工作重复，并帮助您进行更改，使更改成功地被接受到项目中。

* 可以制作小功能，并直接作为Pull提交。

## 提交拉取请求 \(PR\)

在提交Pull请求\(PR\)之前，请考虑以下准则：

* 搜索 [GitHub](https://github.com/amcdnl/ngxs/pulls) 中与您提交的内容有关的开放或关闭的PR

* 创建一个新的git分支:

  ```text
  git checkout -b my-fix-branch master
  ```

* 在本地分支上进行更改。
* 运行完整的测试套件，并确保所有测试均通过。
* 使用我们[commit message guidelines](contributing.md#commit-message-guidelines) 中描述性提交消息来提交更改. 必须遵守这些约定，以保持干净的git日志。
* 将您的分支推送到GitHub:

  ```text
  git push origin my-fix-branch
  ```

* 在GitHub上,向`ngxs:master`发送一个pull request.
* 如果建议更改，那么就需要:
  * 进行所需的更新。
  * 重新运行测试套件，以确保测试仍然通过。
  * 重新设置分支基础并强制推送到GitHub上的库\(这将更新你的 Pull Request\):

    ```text
    git rebase master -i
    git push -f
    ```

就是这样！ 感谢您的贡献！

## 提交消息格式

每个提交消息都由一个 **header**, 一个 **body** 和一个 **footer**组成。 标头具有一种特殊的格式，包括一个 **type**, 一个 **scope** 和一个 **subject**:

```text
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

**header**是必须的，标头的 **scope** 是可选的。

提交消息的任何一行都不能超过100个字符！ 这使得该消息在GitHub以及各种git工具中更易于阅读。

### 还原

如果该提交恢复了先前的提交，则应以`revert:`开头，然后是恢复后的提交的标头。 在主体中应该注明：`This reverts commit <hash>.`，其中哈希是要还原的提交的SHA。

### 类型

必须为以下之一：

* **feat**: 新功能
* **fix**: 错误修复
* **docs**: 仅文档更改
* **style**: 不会影响代码含义的更改\(空格，格式，丢失，分号\)
* **refactor**: 既无法修复错误也未添加功能的代码更改
* **perf**: 更改代码以提高性能
* **test**: 添加缺失的测试
* **chore**: 更改构建过程或辅助工具和库，例如文档生成

### Scope

可以是指定提交更改位置的任何内容。 例如`Store`, `Mutation`, `Action`, `Select`等。

### Subject

包含对变更的简洁描述：

* 使用现在时命令: "change" 而不是 "changed" 或 "changes"
* 不要大写第一个字母
* 末尾不要加点 \(.\)

### Body

和**subject**中一样，使用现在时态的命令："change" 而不是 "changed" 或 "changes"。body应包括改变的动机，并将其与以前的行为进行对比。

### Footer

Footer应包含有关**Breaking Changes**的所有信息，并且也是引用**Closes**的GitHub问题的地方。

**Breaking Changes**应以单词 `BREAKING CHANGE:` 开头，并以空格或两个换行符开头。 然后，将其余的提交消息用于此。

