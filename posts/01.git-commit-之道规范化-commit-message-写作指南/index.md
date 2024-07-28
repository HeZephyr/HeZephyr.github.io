# Git Commit 之道：规范化 Commit Message 写作指南

## commit message 规范

commit message格式都包括三部分：Header，Body和Footer

```
&lt;type&gt;(&lt;scope&gt;): &lt;subject&gt;

&lt;body&gt;

&lt;footer&gt;
```

Header是必需的，Body和Footer则可以省略

### Header

1. Type（必需）

   type用于说明`git commit`的类别，允许使用下面几个标识。

   &gt; * `feat`：新功能（Feature）
   &gt;
   &gt;   &#34;feat&#34;用于表示引入新功能或特性的变动。这种变动通常是在代码库中新增的功能，而不仅仅是修复错误或进行代码重构。
   &gt;
   &gt; * `fix/to`：修复bug。这些bug可能由QA团队发现，或由开发人员在开发过程中识别。
   &gt;
   &gt;   - `fix`关键字用于那些直接解决问题的提交。当创建一个包含必要更改的提交，并且这些更改能够直接修复已识别的bug时，应使用`fix`。这表明提交的代码引入了解决方案，并且问题已被立即解决。
   &gt;   - `to`关键字则用于那些部分处理问题的提交。在一些复杂的修复过程中，可能需要多个步骤或多次提交来完全解决问题。在这种情况下，初始和中间的提交应使用`to`标记，表示它们为最终解决方案做出了贡献，但并未完全解决问题。最终解决问题的提交应使用`fix`标记，以表明问题已被彻底修复。
   &gt;
   &gt; * `docs`：文档（Documentation）
   &gt;
   &gt;   &#34;docs&#34; 表示对文档的变动，这包括对代码库中的注释、README 文件或其他文档的修改。这个前缀的提交通常用于更新文档以反映代码的变更，或者提供更好的代码理解和使用说明。
   &gt;
   &gt; * `style`: 格式（Format）
   &gt;
   &gt;   &#34;style&#34; 用于表示对代码格式的变动，这些变动不影响代码的运行。通常包括空格、缩进、换行等风格调整。
   &gt;
   &gt; * `refactor`：重构（即不是新增功能，也不是修改bug的代码变动）
   &gt;
   &gt;   &#34;refactor&#34; 表示对代码的重构，即修改代码的结构和实现方式，但不影响其外部行为。重构的目的是改进代码的可读性、可维护性和性能，而不是引入新功能或修复错误。
   &gt;
   &gt; * `perf`: 优化相关，比如提升性能、体验
   &gt;
   &gt;   &#34;perf&#34; 表示与性能优化相关的变动。这可能包括对算法、数据结构或代码实现的修改，以提高代码的执行效率和用户体验。
   &gt;
   &gt; * `test`：增加测试
   &gt;
   &gt;   &#34;test&#34; 表示增加测试，包括单元测试、集成测试或其他类型的测试。
   &gt;
   &gt; * `chore`：构建过程或辅助工具的变动
   &gt;
   &gt;   &#34;chore&#34; 表示对构建过程或辅助工具的变动。这可能包括更新构建脚本、配置文件或其他与构建和工具相关的内容。
   &gt;
   &gt; * `revert`：回滚到上一个版本
   &gt;
   &gt;   &#34;revert&#34; 用于回滚到以前的版本，撤销之前的提交。
   &gt;
   &gt; * `merge`：代码合并
   &gt;
   &gt;   &#34;merge&#34; 表示进行代码合并，通常是在分支开发完成后将代码合并回主线。
   &gt;
   &gt; * `sync`：同步主线或分支的Bug
   &gt;
   &gt;   &#34;sync&#34; 表示同步主线或分支的 Bug，通常用于解决因为合并而引入的问题。

2. Scope（可选）

   `scope`用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

   例如修改了`Dao`或者`Controller`，则可以添加表示这些范围受到影响，这有助于更清晰地理解提交的变更影响范围。例如：

   ```bash
   feat(Controller): 添加用户登录功能
   ```

   这个提交消息中，`Controller` 是 `scope`，表示这次提交影响了控制层。

   ```bash
   fix(DataAccess): 修复数据查询逻辑
   ```

   这个提交消息中，`DataAccess` 是 `scope`，表示这次提交影响了数据访问层。

   如果你的修改影响了不止一个scope，你可以使用`*`代替。

3. Subject（必需）

   `subject`是 commit 目的的简短描述，不超过50个字符。规范如下：

   &gt; - 以动词开头，使用第一人称现在时，比如`change`，而不是`changed`或`changes`
   &gt; - 第一个字母小写
   &gt; - 结尾不加句号（`.`）

   例如：

   ```
   feat(UserAuth): implement user authentication
   ```

   这个提交消息中，`implement user authentication` 是 `subject`，简洁明了地描述了引入用户认证功能的目的。

   ```
   fix(Validation): correct input validation logic
   ```

   这个提交消息中，`correct input validation logic` 是 `subject`，清晰地说明了修复输入验证逻辑的目的。

### Body

Body 部分是对本次 commit 的详细描述，可以分成多行。Body编写有两个注意点。

&gt; 1. 使用第一人称现在时，比如使用`change`而不是`changed`或`changes`。这有助于使描述更加直观和连贯，增强可读性。
&gt; 2. 应该说明代码变动的动机，以及与以前行为的对比。 `Body` 部分不仅仅是描述代码的变动，还应该解释为什么进行这个变动，以及与之前的代码行为相比有哪些改进。这有助于其他开发者更好地理解代码变更的背后动机和意图。

### Footer

Footer 部分只用于两种情况。

1. 不兼容变动

   如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法。

2. 关闭 Issue

   如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

   &gt; ```bash
   &gt; Closes #234
   &gt; ```

   也可以一次关闭多个 issue 。

   &gt; ```bash
   &gt; Closes #123, #245, #992
   &gt; ```

### 示例

* 添加用户配置文件编辑功能

  &gt; ```bash
  &gt; feat(UserProfile): add user profile editing feature
  &gt; 
  &gt; This commit introduces a new feature that allows users to edit their profiles
  &gt; directly from the user interface. The motivation behind this change is to
  &gt; enhance user interaction and provide a more seamless experience.
  &gt; 
  &gt; Previously, users had to navigate to a separate editing page to update their
  &gt; profile information. With this new feature, users can now make changes
  &gt; efficiently from their profile page, eliminating unnecessary steps in the
  &gt; workflow.
  &gt; 
  &gt; Changes included in this commit:
  &gt; - Added a new &#39;Edit Profile&#39; button on the user profile page.
  &gt; - Implemented frontend components for profile editing.
  &gt; - Updated backend API to handle profile updates securely.
  &gt; 
  &gt; By streamlining the profile editing process, we aim to improve overall user
  &gt; satisfaction and make our application more user-friendly. This enhancement is
  &gt; in response to user feedback, addressing the need for a more intuitive and
  &gt; accessible way to modify profile details.
  &gt; 
  &gt; Closes #234
  &gt; ```

* 纠正输入验证逻辑

  &gt; ```bash
  &gt; fix(Validation): correct input validation logic
  &gt; 
  &gt; This commit addresses an issue related to input validation logic in the
  &gt; application. Previously, the validation process was not handling certain edge
  &gt; cases correctly, leading to unexpected behavior in specific scenarios.
  &gt; 
  &gt; To resolve this issue, the validation logic has been revised to properly
  &gt; handle various input scenarios. This ensures that user input is thoroughly
  &gt; validated, reducing the likelihood of errors in the application.
  &gt; 
  &gt; The changes made in this commit include:
  &gt; - Correcting boundary checks for user input.
  &gt; - Improving error messages for better user guidance.
  &gt; 
  &gt; These adjustments align with our commitment to delivering a robust and
  &gt; reliable application experience.
  &gt; 
  &gt; Closes #123
  &gt; ```

* 优化数据库查询

  &gt; ```bash
  &gt; refactor(DataAccess): optimize database queries
  &gt; 
  &gt; In this commit, we have refactored the data access layer to optimize database
  &gt; queries and improve overall system performance. The existing query structure
  &gt; was identified as a bottleneck during performance testing, leading to longer
  &gt; response times.
  &gt; 
  &gt; Changes made in this commit:
  &gt; - Reorganized database queries to reduce redundant operations.
  &gt; - Utilized database indexing for faster data retrieval.
  &gt; 
  &gt; By optimizing database queries, we expect to see a significant improvement in
  &gt; system responsiveness and user experience.
  &gt; 
  &gt; Closes #456
  &gt; ```

## git commit 工具

### commitizen

[Commitizen](https://github.com/commitizen/cz-cli)是一个强大的工具，用于撰写合格的 Git 提交消息。使用 Commitizen 可以帮助团队遵循统一的提交消息规范，使提交历史更加清晰和易读。

首先，通过以下命令全局安装 Commitizen：

&gt; ```bash
&gt; npm install -g commitizen
&gt; ```

然后，在项目目录里，运行下面的命令，使其支持 Angular 的 Commit message 格式。

&gt; ```bash
&gt; commitizen init cz-conventional-changelog --save --save-exact
&gt; ```

这个命令会配置项目，使其支持 Angular 规范的 Commit Message。在执行命令时，你可以选择其他预定义的规范或者创建自定义规范。

之后，当你执行 `git commit` 命令时，将其替换为 `git cz`。此时，Commitizen 将引导你通过一个交互式的界面，以生成符合规范的 Commit Message。

![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/bg2016010605.png)
在这个交互式界面中，你可以选择提交的类型（feat、fix、docs 等）、影响的范围（scope）、简短的描述（subject）以及其他相关信息。通过这种方式，可以确保提交消息符合规范，并提供了更多的上下文信息，便于他人理解变更的目的。

使用 Commitizen 和规范化的提交消息格式，有助于提高代码库的可读性，方便生成自动化的变更日志，并促使开发者更注重写出清晰、明确的提交消息。

### commitlint

[commitlint](https://github.com/conventional-changelog/commitlint)是一个用于检查提交消息是否符合指定规范的工具。它可以帮助团队确保 Git 提交消息的一致性和规范性，尤其是当项目采用类似 Angular Commit Message Conventions 的规范时。

1. 安装 Commitlint

   &gt; 首先，你需要安装 `commitlint` 及其相关的配置和规则。通常，`@commitlint/config-conventional` 是与 Angular 规范兼容的配置。
   &gt;
   &gt; ```bash
   &gt; npm install --save-dev @commitlint/config-conventional @commitlint/cli
   &gt; ```

2. 配置 Commitlint

   &gt; 在项目根目录下创建 `commitlint.config.js` 文件，并添加如下内容：
   &gt;
   &gt; ```js
   &gt; module.exports = {
   &gt;   extends: [&#39;@commitlint/config-conventional&#39;],
   &gt; };
   &gt; ```
   &gt;
   &gt; 这个配置文件使用了 `@commitlint/config-conventional` 中预定义的规则，确保符合常见的提交规范。

3. 配置Git钩子

   &gt; 你可以使用 Husky 钩子工具来在提交前运行 `commitlint`。首先，安装 Husky：
   &gt;
   &gt; ```
   &gt; bashCopy code
   &gt; npm install --save-dev husky
   &gt; ```
   &gt;
   &gt; 然后，在 `package.json` 中添加以下配置：
   &gt;
   &gt; ```
   &gt; jsonCopy code
   &gt; &#34;husky&#34;: {
   &gt;   &#34;hooks&#34;: {
   &gt;     &#34;commit-msg&#34;: &#34;commitlint -E HUSKY_GIT_PARAMS&#34;
   &gt;   }
   &gt; }
   &gt; ```
   &gt;
   &gt; 这样配置后，每次提交前都会自动运行 `commitlint` 检查提交消息是否符合规范。

## 生成Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成（[例1](https://github.com/karma-runner/karma/blob/master/CHANGELOG.md)，[例2](https://github.com/btford/grunt-conventional-changelog/blob/master/CHANGELOG.md)）。

![image-20231112222340929](/Users/zfhe/Library/Application Support/typora-user-images/image-20231112222340929.png)

生成的文档包括以下三个部分。

&gt; - New features（新特性）
&gt; - Bug fixes（bug修复）
&gt; - Breaking changes（重大变更）

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。

[conventional-changelog](https://github.com/ajoslin/conventional-changelog) 就是生成 Change log 的工具，运行下面的命令即可。

&gt; ```bash
&gt; npm install -g conventional-changelog
&gt; cd my-project
&gt; conventional-changelog -p angular -i CHANGELOG.md -w
&gt; ```

上面命令不会覆盖以前的 Change log，只会在`CHANGELOG.md`的头部加上自从上次发布以来的变动。

如果你想生成所有发布的 Change log，要改为运行下面的命令。

&gt; ```bash
&gt; $ conventional-changelog -p angular -i CHANGELOG.md -w -r 0
&gt; ```

为了方便使用，可以将其写入`package.json`的`scripts`字段。

&gt; ```javascript
&gt; {
&gt;   &#34;scripts&#34;: {
&gt;     &#34;changelog&#34;: &#34;conventional-changelog -p angular -i CHANGELOG.md -w -r 0&#34;
&gt;   }
&gt; }
&gt; ```

以后，直接运行下面的命令即可。

&gt; ```bash
&gt; $ npm run changelog
&gt; ```

这个自动化流程不仅简化了 Change log 的生成过程，还确保了记录项目变更的一致性和准确性。生成的文档会按照新特性、bug 修复和重大变更等分类，方便用户快速了解每个版本的变更情况。

## 参考资料

1. [如何规范你的Git commit？—阿里云开发者](https://zhuanlan.zhihu.com/p/182553920)
2. [Commit message 和 Change log 编写指南—阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)







---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://lruihao.cn/posts/01.git-commit-%E4%B9%8B%E9%81%93%E8%A7%84%E8%8C%83%E5%8C%96-commit-message-%E5%86%99%E4%BD%9C%E6%8C%87%E5%8D%97/  

