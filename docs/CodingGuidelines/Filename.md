---
sidebar_label: 'GS文件命名规范'
sidebar_position: 1
---

# GS文件命名说明

## 简介

这里主要是展示实施pkg、游戏项目中的文件命名。

## 命名方法

| 名称           | 说明                                     | 示例        |
| -------------- | ---------------------------------------- | ----------- |
| 大驼峰命名法   | 每个单词的首个字母大写                   | HelloWorld  |
| 蛇形命名法     | 每个单词的所有字母都是小写，单词之间用下划线'_'连接 | hello_world |

## 组件脚本

只作为组件的脚本文件称之为组件脚本。组件脚本的命名规则如下：
- 以'F'开头，表明这是一个组件；F应该是feature的意思。
- 后续的单词采用大驼峰命名法。
- 以engine0中的组件脚本命名举例：

```c++
FEntity.gs
FEntityContainer.gs
FLookField.gs
```

## pkg入口脚本

pkg里和pkg同名的脚本文件称之为pkg入口脚本。
pkg入口脚本命名使用蛇形命名法。
- 举例：
```c++
// pkg.cjson入口脚本
cjson.gs

// pkg.worker_thread入口脚本
worker_thread.gs
```

## 游戏项目中的文件命名
除了上面提到的文件命名之外，游戏项目中还会根据功能或者需求细分一些脚本文件命名。以下列出的命名都以engine0为例。

### daemon脚本
- 单词采用大驼峰命名法。
- 最后增加'D'为结束；'D'应该是daemon的意思。
- 以engine0中的daemon举例(/engine/daemons/)：

```c++
SystemD.gs
LoginQueueD.gs
```

### mod入口脚本
和mod同名的脚本文件称之为mod入口脚本。
mod入口脚本命名规则使用蛇形命名法。
  - 以engine0中的mod举例(/engine/mods/)：

```c++
// engine.mods.net入口脚本
net.gs

// engine.mods.log_analysis入口脚本
log_analysis.gs
```

## 其他脚本
- 其他未做出明确说明的脚本文件的命名，默认使用大驼峰命名法命名。以engine0中举例：
```c++
User.gs
ScriptUtil.gs
```

- 特别的，出于功能或者需求需要时，根据实际情况命名。以engine0举例：
```c++
// 在engine0的/cmds/里，用来处理玩家登录指令的脚本文件，
// 要求脚本文件名和登录指令同名

// 登录指令
login.gs

// gm指令
gm.gs
```