# Git使用规范

* [分支管理规范](#分支管理规范)
    - [分支说明和操作](#分支说明和操作)
    - [项目分支操作流程示例](#项目分支操作流程示例)
* [提交信息规范](#提交信息规范)
* [参考资料](#参考资料)

## 分支管理规范

### 分支说明和操作

- master 分支

    主分支，永远处于稳定状态，对应当前线上版本

    以 tag 标记一个版本，因此在 master 分支上看到的每一个 tag 都应该对应一个线上版本

    不允许在该分支直接提交代码

- develop 分支

    开发分支，包含了项目最新的功能和代码，所有开发都依赖 develop 分支进行

    小的改动可以直接在 develop 分支进行，改动较多时切出新的 feature 分支进行

    注： 更好的做法是 develop 分支作为开发的主分支，也不允许直接提交代码。小改动也应该以 feature 分支提 merge request 合并，目的是保证每个改动都经过了强制代码 review，降低代码风险

- feature 分支

    功能分支，开发新功能的分支

    开发新的功能或者改动较大的调整，从 develop 分支切换出 feature 分支，分支名称为 feature/xxx

    开发完成后合并回 develop 分支并且删除该 feature/xxx 分支

- release 分支

    发布分支，新功能合并到 develop 分支，准备发布新版本时使用的分支

    当 develop 分支完成功能合并和部分 bug fix，准备发布新版本时，切出一个 release 分支，来做发布前的准备，分支名约定为release/xxx

    发布之前发现的 bug 就直接在这个分支上修复，确定准备发版本就合并到 master 分支，完成发布，同时合并到 develop 分支

- hotfix 分支

    当线上版本出现 bug 时，从 master 分支切出一个 hotfix/xxx 分支，完成 bug 修复，然后将 hotfix/xxx 合并到 master 和 develop 分支(如果此时存在 release 分支，则应该合并到 release 分支)，合并完成后删除该 hotfix/xxx 分支

### 项目分支操作流程示例
- 切到 develop 分支，更新 develop 最新代码
    ```
    git checkout develop
    ​git pull --rebase
    ```
- 新建 feature 分支，开发新功能
    ```
    git checkout -b feature/xxx
    ​...
    ​git add <files>
    ​git commit -m "feat(xxx): commit a"
    ​git commit -m "feat(xxx): commit b"
    ​# 其他提交...
    ```

    如果此时 develop 分支有一笔提交，影响到你的 feature 开发，可以 rebase develop 分支，前提是 该 feature 分支只有你自己一个在开发，如果多人都在该分支，需要进行协调：

    ```
    # 切换到 develop 分支并更新 develop 分支代码
    ​git checkout develop
    ​git pull --rebase
    ​
    ​# 切回 feature 分支
    ​git checkout feature/xxx
    ​git rebase develop
    ​
    ​# 如果需要提交到远端，且之前已经提交到远端，此时需要强推(强推需慎  重！)
    ​git push --force
    ```

    上述场景也可以通过 git cherry-pick 来实现，有兴趣的可以去了解一下这个指令。
- 完成 feature 分支，合并到 develop 分支
    ```
    # 切到 develop 分支，更新下代码
    ​git check develop
    ​git pull --rebase
    ​
    ​# 合并 feature 分支
    ​git merge feature/xxx --no-ff
    ​
    ​# 删除 feature 分支
    ​git branch -d feature/xxx
    ​
    ​# 推到远端
    ​git push origin develop
    ```

- 当某个版本所有的 feature 分支均合并到 develop 分支，就可以切出 release 分支，准备发布新版本，提交测试并进行 bug fix
    ```
    # 当前在 develop 分支
    ​git checkout -b release/xxx
    ​
    ​# 在 release/xxx 分支进行 bug fix
    ​git commit -m "fix(xxx): xxxxx"
    ...
    ```

- 所有 bug 修复完成，准备发布新版本
    ```
    # master 分支合并 release 分支并添加 tag
    ​git checkout master
    ​git merge --no-ff release/xxx --no-ff
    ​
    ​# 添加版本标记，这里可以使用版本发布日期或者具体的版本号
    ​git tag 1.0.0
    ​
    ​# develop 分支合并 release 分支
    ​git checkout develop
    ​git merge --no-ff release/xxx
    ​
    ​# 删除 release 分支
    ​git branch -d release/xxx
    ```

- 线上出现 bug，需要紧急发布修复版本
    ```
    # 当前在 master 分支
    ​git checkout master

    ​# 切出 hotfix 分支
    ​git checkout -b hotfix/xxx

    ​... 进行 bug fix 提交

    ​# master 分支合并 hotfix 分支并添加 tag(紧急版本)
    ​git checkout master
    ​git merge --no-ff hotfix/xxx --no-ff

    ​# 添加版本标记，这里可以使用版本发布日期或者具体的版本号
    ​git tag 1.0.1

    ​# develop 分支合并 hotfix 分支(如果此时存在 release 分支的话，应   当合并到 release 分支)
    ​git checkout develop
    ​git merge --no-ff hotfix/xxx

    ​# 删除 hotfix 分支
    ​git branch -d hotfix/xxx
    ```

## 提交信息规范
    <type>(<scope>): <subject>各个部分的说明如下：
- type 类型，提交的类别
    - feat：新功能
    - add：新加功能或文件
    - delete：删除功能或文件
    - fix：修复 bug
    - merge：合并代码
    - docs：文档变动
    - style：格式调整，对代码实际运行没有改动，例如添加空行、格式化等
    - refactor：bug 修复和添加新功能之外的代码改动（代码重构）
    - review：代码审核
    - perf：提升性能的改动
    - test：添加或修正测试代码
    - chore：构建过程或辅助工具和库（如文档生成）的更改

- scope 修改范围

    主要是这次修改涉及到的部分，简单概括，例如 login、train-order

- subject 修改的描述

    具体的修改描述信息
    
- 范例

    feat(detail): 详情页修改样式
    
    fix(login): 登录页面错误处理

    test(list): 列表页添加测试代码

## 参考资料
https://www.cnblogs.com/lucky-t/archive/2020/12/28/14200360.html