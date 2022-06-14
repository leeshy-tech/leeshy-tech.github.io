# git使用笔记——commit格式


> 起因是看到了三位大佬的工程https://github.com/Direktor799/rusted_os/commits/main，非常的赏心悦目，才知道commit也有固定的格式

Commit Message格式，目前规范使用较多的是 [Angular 团队的规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines), 继而衍生了 [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/). 很多工具也是基于此规范, 它的 message 格式如下:

Commit格式包含三个部分，Header、Body、Footer

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

- 首行header：必填，描述修改类型和内容
  - scope：commit影响的范围
  - subject：commit的概述
- body：commit具体修改的内容
- footer：备注，通常是 BREAKING CHANGE 或修复的 bug 的链接.

### Header

Header只有一行，包括三个字段`type`（必填）`scope`（可选）`subject`（必填）

#### type

说明commit的类别

- feat：新增功能
- fix：bug 修复
- docs：文档更新
- style：不影响程序逻辑的代码修改(修改空白字符，格式缩进，补全缺失的分号等，没有改变代码逻辑)
- refactor：重构代码(既没有新增功能，也没有修复 bug)
- perf：性能, 体验优化
- test：新增测试用例或是更新现有测试
- build：主要目的是修改项目构建系统(例如 glup，webpack，rollup 的配置等)的提交
- ci：主要目的是修改项目继续集成流程(例如 Travis，Jenkins，GitLab CI，Circle等)的提交
- chore：不属于以上类型的其他类，比如构建流程, 依赖管理
- revert：回滚某个更早之前的提交

#### scope

说明commit影响的范围，比如文件或者逻辑层。

#### subject

commit简述

- 以动词开头，使用第一人称现在时，比如`change`，而不是`changed`或`changes`
- 第一个字母小写
- 结尾不加句号（`.`）

### Body

commit详细描述

### Footer

只用于两种情况

1. 不兼容变动
2. 关闭Issue
