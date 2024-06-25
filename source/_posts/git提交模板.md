---
title: git提交模板
tags:
  - git
  - vscode
abbrlink: fb89d8
date: 2024-06-25 08:40:27
---

>提交模板基于**Angular**项目提交规范

参考vscode的 `git-commit-plugin`插件

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240625/20240625083948.png)

# 日志结构

```shell
Header
<空一行>
Body
<空一行>
Footer
```

具体内容：

```shell
Type(Scope): Subject

Body

Footer
```

# 符号说明

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240619/20240619124503.png)

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240619/20240619124516.png)


# 示例

```
commit 3c12bcbc95b037396268e505791387a23f63da17
Author: juzhango <luozw888@outlook.com>
Date:   Wed Jun 19 12:38:37 2024 +0800

    🐞 fix(括号内填涉及范围Scope): 这里写概述Subject内容

    1.这里写Body内容
    2xxx
    3xxx

    备注Footer内容可以写相关单号
```

短日志

```shell
3c12bcb 🐞 fix(括号内填涉及范围): 这里写概述(Subject内容)
```

实际项目短日志

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240625/20240625085701.png)


template:

```
[🎉 init|✨ feat|🐞 fix|📃 docs|🌈 style|🦄 refactor|🎈 perf|🧪 test|🔧 build|↩ revert](Scope): Subject

Body:

Footer:

```

# 当前使用的

```shell
[🎉 init|✨ feat|🐞 fix|📃 docs|🌈 style|🦄 refactor|🎈 perf|🧪 test|🔧 build|↩ revert](改动涉及模块): 简要描述

【修改内容】: 必填项
【问题原因】: 可选项
【解决方案】: 可选项
【需要测试】: yes/no

【相关单号】: -

```
