---
sidebar_label: '提交规则'
sidebar_position: 2.3
---

# 提交规则

为了方便通过git提交记录，提取更新内容，自动生成release notes，需要统一git commit的提交规范

---------------

## 变更类型

变更分为以下几类：

| 前缀 | 意义 |
| ---- | ---- |
| BUG | 修复BUG，例如：修复XXX导致的崩溃，XXX的结果不符合预期等 |
| NEW | 新功能，例如：添加XXX函数，添加XXX类型等 |
| FIX | 行为调整，既不属于BUG也不属于NEW的一些变动，但会影响相关功能的使用，例如：XXX函数返回值类型调整；重命名XXX接口等 |
| MINOR | 微小改动，以及不会影响使用者的变动，例如：整理代码；调整cicd流程等

---------------

## 提交信息编写规则

向gitlab上提交代码时，提交信息(commit-msg)应当遵循以下规则：

* 一次提交(commit)中可以有若干个`变更记录`，因此某次提交的提交信息可能是这样的：

```
* BUG : A problem will make driver crash.
- xxx.ptr may be nullptr.

* FIX : efun 'get_xxxxx' will return nil when failed now.

* FIX : add '/' after path get by 'get_xxxx_path'

* MINOR : remove useless return.
- in 'cmm_xxxx_xxxxx.cpp'
```

* 一个合格的`变更记录`看起来会是这样的：

```
*【类型】:【提交信息的标题】
-【可选的说明：对提交信息的一些补充说明】
- ...
```

* `变更记录`由 '`*`'(可选)和提交类型开头，然后是'`:`'(可选，推荐加上)后面跟着这份提交信息的标题。

* 其中，`类型`可以是 `BUG/NEW/FIX/MINOR` 其中之一；如果认为只有标题难以说明，可以另起一行以 '`-`' 开头，进行补充。

* 如果认为某次变更非常重要，可以在类型前加 '`!`'，'`!`' 的数量越多，说明这份变动越重要（目前最多两个，超过两个的等于两个）：

```
* !BUG : fix a very big bug.

* !!FIX: API changed: old one will cause crash!
```

* 变更的重要程度影响其在Release-Note上的排序权重（也会影响颜色XD）

---------------

## 版本命名规则
一个合规的版本号应该是这样的：

    major.minor.patch[-build[-name]]

以下是合法的版本号
* 1.0.1
* 1.0.1-12092101
* 10.0.5-12092115-alpha

以下是不合法的版本号
* 1.0 
* 2.0.6-stable
* 1.1.5-12521abc

---------------
## 提交信息检查及Release-Note生成流程

* 当push/tags动作触发时，cicd会调用一个python脚本，找到base节点，从当前分支的head开始，对中间的每个提交的commit-msg执行语法检查，并报告遇到的问题。
  * 如果执行的是push操作，base节点是当前分支与master的最佳公共节点（merge-base）；如果当前分支就是master，则base节点是当前提交链上最近的tags
  * 如果执行的是tags操作，则base节点是当前提交链上最近的合法版本tags（见版本命名规则）
  * 如果提交中存在语法错误，脚本会报告当前错误的行号、列号、原因以及错误发生在哪次提交,如下所示：

```bash
Error at (1, 5) Error commit type: should be 'NEW', 'FIX', 'BUG' or 'MINOR'.
 in commit:  Fixed bug when fail to parse arguments
  ( 1f3f1ea30cb0e3d7df489938e4f2f7a5bf37fb46 )
```

* 当执行tags动作时，除了检查提交信息，还会检查当前tags-name是不是一个合规的版本号，如果是，则执行Release-Note的生成动作，Release-Note包含以下信息：
  * 当前版本号
  * 之前的版本（可能有若干个）
  * 根据commit-msg生成的更新内容
  
* P.S. 目前生成出来的Release-Note **暂时** 被放在 `\\cnszvwin0007\everyone\xuxr2\gs-release-notes`中。

---------------
## 提交检查相关配置

这部分信息主要面向cicd的配置，方便其他项目也使用这份脚本进行提交检查

负责检查提交的cicd配置如下所示：

```yml
verify-commit-message-on-branch:
  except:
    - schedules
    - master
    - tags
  script:
    # - source ./cicd/check_gitpython.sh      #for unix
    - call cicd/check_gitpython.bat           #for windows
    - python cicd/python_scripts/verify_commit_msg.py 
      --current-branch-name %CI_COMMIT_REF_NAME% 
      --aim-branch-name master
      --max-commit 100
      --accept-non-ascii-ch 0
      --allow-invalid-commit-msg 0
      --git-project-name %CI_PROJECT_NAME%
      --git-project-url %CI_PROJECT_URL% 
  stage: verify_commit
  tags:
    - win7gs_cmake

verify-commit-message-on-master:
  except:
    - schedules
  only:
    - master
  script:
    # - source ./cicd/check_gitpython.sh      #for unix
    - call cicd/check_gitpython.bat           #for windows
    - python cicd/python_scripts/verify_commit_msg.py
      --current-branch-name %CI_COMMIT_REF_NAME% 
      --aim-branch-name master
      --accept-non-ascii-ch 1 
      --allow-invalid-commit-msg 1
      --git-project-name %CI_PROJECT_NAME%
      --git-project-url %CI_PROJECT_URL% 
      # ATTENTION: allow non-ascii character & invalid commit-message for temp
  stage: verify_commit
  tags:
    - win7gs_cmake

verify-commit-message-on-tags:
  except:
    - schedules
  only:
    - tags
  script:
    # - source ./cicd/check_gitpython.sh      #for unix
    - call cicd/check_gitpython.bat           #for windows
    - python cicd/python_scripts/verify_commit_msg.py
      --current-branch-name %CI_COMMIT_REF_NAME% 
      --current-tags %CI_COMMIT_TAG% 
      --accept-non-ascii-ch 1
      --allow-invalid-commit-msg 1
      --git-project-name %CI_PROJECT_NAME%
      --git-project-url %CI_PROJECT_URL%
      --release-notes-output-path //cnszvwin0007/everyone/xuxr2/gs-release-notes
      # ATTENTION: allow non-ascii character & invalid commit-message for temp
  stage: verify_commit
  tags:
    - win7gs_cmake
```

* 在调用 `cicd/python_scripts/verify_commit_msg.py` 开始检查之前，先调用`cicd/check_gitpython.bat`保证`gitpython`已经安装。
* `cicd/python_scripts/verify_commit_msg.py` 使用一系列命令行参数来指定其行为：

    * `--current-branch-name <CURRENT_BRANCH_NAME>` 
      * 指定当前分支名，默认值为 `now_repo.active_branch.name` 但是在cicd上默认值可能无效，所以需要手动指定
    * `--aim-branch-name <AIM_BRANCH_NAME>` 
      * 指定目标分支名，默认值为 `"master"`
      * 目标分支通常被指定为主分支，检查时将会找到当前分支与目标分支的公共提交节点作为检查的终点，若目标分支与当前分支相同，则检查的终点为距离最近的若干个带有合法版本号的tags
      * 当 `--current-tags` 被指定时，此命令行无效
    * `--current-tags <CURRENT_TAGS>`
      * 指定标签，默认值为 `""`
      * 通常在执行tags动作时指定，将会检查指定的tags名称是否合法，若合法，则从当前提交头开始，统计到最近若干个合法tags，其间的所有提交信息将被用于生成Release-Note；若指定的tags名称不是一个合法的版本号，则报告 `"Invalid version-tag, abort"` 然后终止
    * `--max-commit <NUM>`
      * 指定最大允许的提交数，默认值为 `10`
      * 仅当 `current-branch-name` 和 `aim-branch-name` 不同时生效；计算当前提交节点与两个分支的公共节点之间的提交数，若超过提交数则报告 `"Fail: The maximum number of submissions allowed in a single merge is '", max_commit, "', please combine some commits."` 然后终止。
    * `--min-commit-len <NUM>`
      * 指定变更信息标题所需的最小长度，默认值为 `0`
      * 若一份变更记录的标题字符（中文字符算一个字符）数少于 `min-commit-len` 则报告错误 `" 'COMMIT_TITLE' is too short, 'min_commit_len' is XXX"`
    * `--accept-non-ascii-ch <1/0>`
      * 指定提交信息是否接受非asc-ii字符，默认值为 True
    * `--git-project-name <PROJECT_NAME>`
      * 指定项目名，默认值为 `""`
    * `--git-project-url <PROJECT_URL>`
      * 指定项目所在的仓库页面路径，默认值为 `""`
      * P.S. 项目名和仓库页面路径用于Release-Note生成；项目名被用于生成的Release-Note标题和文件名， 仓库页面路径用于生成Release-Note中更新项目的链接
    * `--release-notes-output-path <PATH>`
      * 指定生成的Release-Note文件存放位置，默认值为 `""`
      * 若路径未被指定，则不生成文件，仅在终端输出生成的Release-Note信息
    * `--allow-invalid-commit-msg <1/0>`
      * 指定遇到非法的提交信息时，是否允许通过检查，默认值为 `False`
