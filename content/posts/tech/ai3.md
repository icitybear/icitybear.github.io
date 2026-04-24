---
title: "Rule Skill Hook" #标题
date: 2026-04-23T10:21:37+08:00 #创建时间
lastmod: 2026-04-23T10:21:37+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- ai
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论 有自带的扩展成twikoo
showToc: true # 显示目录 文章侧边栏toc目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---

# Rule规则
![alt text](image3.png)

这里提供一个设置中文回答 Rule，也可以让你的 Agent 帮你设置全局的中文回答 Rule 并自己安装 
- Cursor 
  - 保存 .md -> ~/.cursor/rules/
- Kiro
  - 保存 .md -> ~/.kiro/steering/
  - ⚠️  注意，Kiro 的 Rules 为 Steering
- Cluade
  - 保存 .md -> ~/.claude/CLAUDE.md 规则放到一个新的md, 然后CLAUDE.md通过@引入
  - Kiro 的中文响应规范。（language配置）Claude Code 对应的配置方式不太一样，它使用 CLAUDE.md 文件来存放指令
  
## kiro的steering
 ~/.kiro/steering/chinese-response.md内容
```
---
description: always
Apply: true 
---
# 语言设置 

## 全局语言规则 
**重要：所有与用户的交互必须使用中文。**

### 适用范围 
- 所有回答和响应 
- 代码注释（除非是英文代码库的标准注释） 
- 文档和说明 
- 错误信息和提示 
- 任务描述和进度报告 

### 例外情况 
- 代码本身（变量名、函数名、类名等保持英文或原有命名规范） 
- 技术术语可以保留英文，但需要在首次出现时提供中文解释 
- 第三方库和框架的名称保持原文 
- 命令行指令和代码片段

### 示例 
✅正确： 

- "我会帮你创建一个新的视图控制器"
- "这个方法使用了 AFNetworking 进行网络请求" 
- "运行‘pod install、来安装依赖"

❌错误： 
- "I will help you create a new view controller" 
- "This method uses AFNetworking for network requests"

```typescript
// ✅好的做法：使用中文注释 
// 获取用户信息 
const getUserInfo = async (userId: string) => {
    // 验证用户ID是否有效 
    if (!userId) {
        throw new Error('用户ID不能为空'); 
    }
    //... 
};

// ❌不好的做法：使用英文注释
// Get user information
const getUserInfo = async (userId: string) => {
    // Validate user ID 
    if (!userId) {
        throw new Error('User ID cannot be empty'); 
    }
    //... 
};

```

## claude.md
  - ~/.claude/CLAUDE.md — 只做索引，通过 @ 引入各规则文件
  - ~/.claude/chinese-response.md — 中文响应规范
  - ~/.claude/RTK.md — 已有的规则
``` markdown
@RTK.md
@chinese-response.md
@code-style.md
@code-review.md
```
-  ~/.claude/chinese-response.md
``` markdown
# 语言设置

## 全局语言规则
**重要：所有与用户的交互必须使用中文。**

### 适用范围
- 所有回答和响应
- 代码注释（除非是英文代码库的标准注释）
- 文档和说明
- 错误信息和提示
- 任务描述和进度报告

### 例外情况
- 代码本身（变量名、函数名、类名等保持英文或原有命名规范）
- 技术术语可以保留英文，但需要在首次出现时提供中文解释
- 第三方库和框架的名称保持原文
- 命令行指令和代码片段
```

# hooks
1. settings.json 里内联 — 适合简单的单行命令
2. .claude/hooks/ 目录下独立脚本 — 适合复杂逻辑，更清晰
settings.json文件 项目级别 settings.local.json
``` json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read -r f; [[ \"$f\" == *.go ]] && gofmt -w \"$f\"; } 2>/dev/null || true",
            "timeout": 10,
            "statusMessage": "Formatting Go file..."
          }
        ]
      },
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/proto-client-gen.sh",
            "timeout": 30,
            "statusMessage": "Generating proto client code..."
          }
        ]
      }
    ]
  }
}
```
- .claude/hooks/proto-client-gen.sh 对应sh脚本
``` python
#!/bin/bash
# Proto Client 代码生成
# 当 api 目录下的 .proto 文件被编辑时，自动执行 kratos proto client 生成 Go 代码

file_path=$(jq -r '.tool_input.file_path')

if [[ "$file_path" == */api/*.proto ]]; then
  kratos proto client "$file_path"
fi
```

这是 Claude Code hook 和 Kiro hook 的关键区别： 
- Kiro hook 的 fileEdited 监听的是 IDE 里的文件保存事件
- Claude Code hook 的 PostToolUse 只在 Claude 调用工具后触发 所以如果你是在 IDE 里手动编辑的 proto 文件，hook 不会执行。只有当你让 Claude 帮你编辑 proto 文件时才会自动触发。 手动编辑后你需要自己跑一下： kratos proto client api/media/media.proto

kiro的情况 
- .kiro/hooks/proto-client-gen.kiro.hook
```json
{
  "name": "Proto Client 代码生成",
  "version": "1.0.0",
  "description": "当 api 目录下的 .proto 文件被编辑保存时，自动执行 kratos proto client 命令生成对应的 Go 代码",
  "when": {
    "type": "fileEdited",
    "patterns": [
      "api/**/*.proto"
    ]
  },
  "then": {
    "type": "askAgent",
    "prompt": "刚才有一个 api 目录下的 .proto 文件被保存了，请找到该文件的路径，然后执行 `kratos proto client <该文件路径>` 命令来生成 Go 代码。只需要执行命令，不需要额外解释。"
  }
}
```

# Skills 是什么
Skills 是由指令、脚本与资源组成的标准化文件夹，Agent 可自动发现并动态加载，以在特定任务上表现得更专业、更可靠。

Agent Skills 是 Anthropic 提出的一套标准化、可组合、可移植、可动态加载的能力封装框架，用于强化 AI Agent 在特定任务中的执行能力。它是一种可复用、结构化的能力包，将指令、脚本与资源统一封装，把专业知识、执行流程、代码逻辑打包为独立 “技能模块”。这些模块内置了领域知识、工作流、脚本与最佳实践，支持 Agent 按需加载、即用即抛，而非始终驻留在上下文里，从而更高效地完成复杂任务。

- Agent = 智能体（能自主思考、行动的 AI，比如能自己查资料、写代码、做规划的 AI）
- Skill = 技能
- Agent Skill = 给 AI 智能体 “装上专业能力 / 工具包”

# 为什么要用 Skills
每次用 AI 写业务代码，是否都有这样的崩溃时刻？
- 重复灌输规范，效率极低 
  - 每次让 AI 写代码，都要从头粘贴一堆基础要求：遵守既定代码规范 这些重复工作占用大量时间，却毫无增值。
  - 无法使用现有框架、组件，接入说明、示例不可能每次都重复，更无法使用 Rules 占用上下文
- Prompt 换个项目就失效
  - 之前调试好的指令，在另一个代码库上完全没用。
- 团队输出风格混乱
  - 不同成员使用的 Prompt 差异极大：有人只简单描述需求，有人则贴满各类规范，导致 AI 生成的代码风格参差不齐，后续整合和维护成本陡增。
  
# Skills 的能力
- 把「重复 Prompt」变成「可复用的公共函数 / SDK」
  - 我们写代码不会把工具逻辑、常量、规范到处复制粘贴，而是封装成公共模块、工具类、SDK。
  - Agent Skills 就是把代码规范、技术栈约束、业务规则等固化为内置能力，像调用封装好的方法一样直接使用，无需重复 “教” AI，避免重复造轮子。
- 解决「Prompt 不可沉淀、不可维护」的问题
  - 以前的 Prompt 是散的、临时的、跟对话绑定的。Agent Skills 是中心化、可版本、可迭代的：更新一次代码规范，所有用到这个技能的 AI 调用自动生效，就像更新公共依赖包。
- 实现团队「标准化」，消除协作混乱
  - Skills 固化 AI 输出标准、业务约束为统一技能。所有人调用同一个技能，输出质量稳定可控，不会因为 “谁写的 Prompt 不一样” 就忽高忽低。
- 让 AI 从「聊天助手」变成「可工程化集成的开发 Agent」
  - Skills 就是给 AI 封装了一套「团队级 SDK + 规范脚手架」 
- 降低上下文损耗
  - 无需携带大段 Rules 规则占用上下文、消耗 token，仅按需加载技能元数据，只需聚焦业务需求即可。

一句话总结：Agent Skills 就是把对 AI 的「临时口头调教」，变成了开发者熟悉的「模块化、可复用、可维护、可协作的工程化能力」。

# Skills 格式
一个 Skill 本质是一个标准化文件夹，包含固定结构与文件（SKILL.md 文件），这是其可移植、可组合的基础。SKILL.md 文件包含元数据（至少包括 name 和 description）以及告诉 Agent 如何执行特定任务的指令。Skill 还可以捆绑脚本、模板和参考材料。

## 标准目录结构
``` 
your-skill-name/
├── SKILL.md           # 【必须】核心文件，技能说明书（含元数据 + 指令）（固定命名）
├── scripts/           # 【可选】可执行脚本（Python/Shell/JS等）
├── assets/            # 【可选】资源文件，用于输出的模板、图标、字体等）
└── references/        # 【可选】文档、参考资料、示例等（按需加载）
```

## Skill 文件夹命名：
  - 使用 kebab-case：your-skill-name ✅
  - 不使用空格：Your Skill Name ❌
  - 不使用下划线：your_skill_name ❌
  - 不使用驼峰大小写：YourSkillName ❌

- 核心文件：**SKILL.md**（关键）
  - 必须完全命名为 SKILL.md（区分大小写）✅
  - 不接受任何变体（SKILL.MD、skill.md 等）❌
  - 必须包含两部分：YAML 元数据（Frontmatter） + Markdown 指令

## YAML 元数据（必需）：
```
---
name: skill-name   # 必须：技能名称
description: A clear description of what this skill does and when to use it 
# 必须：技能描述（用于 Agent 匹配任务，应包含具体使用场景）
--- 

```

字段要求：name（必须）：
- 仅使用 kebab-case
- 无空格或大写字母
- 应与文件夹名称匹配

description（必须）：
```
[它做什么] + [何时使用] + [核心能力]
```
- 必须同时包含：
  - 该 Skill 的功能
  - 何时使用它（触发条件）
- 少于 1024 个字符
- 无 XML 标签（< 或 >）

## Markdown 内容：
- 技能说明、使用方法、行为定义等
- 应保持简洁
- 详细内容放在 references/ 中

## 触发方式
- 当任务匹配技能描述时，自动加载技能内容
- 手动调用：支持通过 /skill-name 命令手动触发或者简单的自然语言

## 渐进式加载机制
Skills 使用三级加载系统来高效管理上下文，确保 Agent 在拥有强大能力的同时不会因信息过载而变得迟钝：

1. Discovery（发现）：启动仅加载技能名称和描述，快速匹配任务相关性，避免占用上下文；
2. Activation（激活）：当任务与某个 Skill 的描述匹配时，按需加载对应技能的完整指令至上下文；
3. Execution（执行）：遵循指令执行任务，按需加载引用文件 / 运行脚本代码。
![alt text](image1.png)
关键原则：
- 保持 SKILL.md 简洁，只保留核心流程与执行指导，不存放冗余细节、扩展说明 （建议 < 5k 词）
- 所有非核心的详细内容，统一移至 references/ 目录文件，references/ 内容与 SKILL.md 保持一级直接关联
- 使用链接引用详细内容，明确告知读者内容位置与使用时机
Skill  vs  MCP  vs  Rules
![alt text](image2.png)



# 工程实践 Skill例子
skills 文件位置
- 项目级 skills：项目根目录/.agent/skills/
  - Cursor 项目根目录/.cursor/skills/ 或 项目根目录/.cursor/skills-cursor/
  - Kiro 项目根目录/.kiro/skills/
  - Claude  /.claude/skills/
- 全局 skills：~/.agent/skills/
  - Cursor ~/.cursor/skills/  或 ~/.cursor/skills-cursor/
  - Kiro ~/.kiro/skills/ 
  - Claude Code  ~/.claude/skills/

## 安装 Skill
- 手动放置到 agent 的 skills 文件夹位置
  - 适用于开发者，自定义 Skill
  - 后期可通过 git 同步不同 Agent 
- OpenSkills 同步安装 (适用不同的agent) 支持这种
  - GitHub https://github.com/numman-ali/openskills
  - 适用于直接使用 GitHub 上的开源 Skill 
```
npx openskills install anthropics/skills // 加载Anthropic Marketplace  GitHub 仓库 本地文件路径 私人 Git 仓库
npx openskills sync // 更新 list搜索 read加载 update更新 remove移除 

npx openskills install anthropics/skills
npx openskills read skill-creator
```


## 开始你的第一个自定义 Skill
skill使用案例
``` markdown
---
name: property-counter
description: 计算并列出 Objective-C 文件中的属性实例数量。触发关键词：jssx
---

# 属性实例计数器

## 何时使用

- 用户输入「jssx」+ 文件名或路径

## 功能说明

分析 Objective-C 文件，统计并列出：
1. `@property` 声明的属性数量
2. 实例变量（`@interface` 中花括号内的变量）
3. 每个属性的详细信息（类型、名称、修饰符）

## 统计规则

### 包含的属性类型
- `@property` 声明的属性
- `@interface` 扩展中的属性
- 私有扩展（匿名 category）中的属性
- 实例变量（ivar）

### 不包含的内容
- 类方法和实例方法
- 局部变量
- 方法参数
- 静态变量

## 输出格式

```markdown
## 属性实例统计：[文件名]

### 总计
- 属性总数：X 个
- 公开属性：X 个
- 私有属性：X 个
- 实例变量：X 个

### 详细列表

#### 公开属性（@interface）
1. `@property (nonatomic, strong) UILabel *titleLabel`
2. `@property (nonatomic, copy) NSString *userName`
...

#### 私有属性（@interface 扩展）
1. `@property (nonatomic, strong) UIButton *closeButton`
2. `@property (nonatomic, assign) BOOL isLoading`
...

#### 实例变量
1. `NSInteger _count`
2. `BOOL _isVisible`
...

### 分析说明
- [可选：对属性数量的评价，如是否过多、是否符合单一职责原则等]
```

安装并触发
![alt text](image4.png)

# 通过 AI 创建 Skill
skill-creator是 Anthropics 官方提供的 skill，用于帮助用户快速创建、编辑和打包自定义 Skill。即一个“用 Skill 生成 Skill”的工具
通过 Anthropics 官方提供的 skill skill-creator创建 skill

- 安装skill-creator[ https://github.com/anthropics/skills/tree/main/skills/skill-creator ]
```
npx openskills install anthropics/skills
npx openskills read skill-creator
```

# 拓展
- Claude 官方提供的 Skills **https://github.com/anthropics/skills/tree/main/skills**
- skills.sh 排行榜：可以直观查看当前最受欢迎的 Skills 仓库和单个 Skill 的使用情况。

# 总结
引入 Skill 对 工程的影响
- 统一 AI 编码行为，严格遵循项目既定代码规范、组件规范与工程规约，确保输出符合团队标准。
- 解决团队协作中 AI 产出质量参差不齐的问题，稳定并统一 AI 生成代码的质量与风格。
- 基于零代码 + 自然语言交互，增强 AI 对复杂组件、业务模块与工程结构的理解与编码能力。