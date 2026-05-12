---
name: guide-generator
description: 为项目生成全景技术指导文档（guide.html）。当用户说"生成指导文档"、"项目指南"、"guide"、"上手指南"、"项目文档"时触发。与 deepwiki 不同，本 skill 生成的是面向开发者上手和维护的指导型文档，不是 API 参考手册。
---

TRIGGER when:
  - 用户说 "生成指导文档", "项目指南", "生成guide", "上手指南", "项目文档", "写个guide"
  - 用户说 "generate project guide", "create guide.html", "project documentation"
DO NOT TRIGGER when: user asks about single module API docs (use deepwiki) or system architecture only (use deepwiki-system)

# Guide Generator — 项目全景技术指导文档生成器

## 定位

**本 skill 生成的是"项目指导文档"，不是"API 参考手册"。**

| 维度 | guide-generator (本 skill) | deepwiki-module |
|------|---------------------------|-----------------|
| 目标读者 | 新接手项目的开发者 | 需要 API 细节的开发者 |
| 核心问题 | "这个项目是做什么的？怎么上手？出了问题怎么办？" | "这个类有哪些方法？参数是什么？" |
| 内容风格 | 指导型（上手 + 排障） | 参考型（API 手册） |
| 知识来源 | 业务逻辑 + 代码结构 | 代码结构为主 |
| 输出 | 单个 guide.html | .md + .html (12章) |

## 生成流程

### Phase 1: 项目理解（必须，不可跳过）

**在写任何内容之前，必须先理解项目。** 禁止不读代码就生成文档。

1. **扫描项目结构**
   - 使用 Glob 扫描目录结构
   - 识别入口文件（main.c, main.py, index.ts, App.java 等）
   - 识别配置文件（config.h, .env, settings.py, package.json 等）
   - 识别文档文件（README.md, doc/, docs/ 等）

2. **读取关键源文件**
   - 入口文件（理解启动流程）
   - 配置文件（理解编译选项/运行参数）
   - 头文件/接口文件（理解模块边界）
   - 设计文档（如有）
   - 至少读取 5 个核心文件

3. **识别项目类型**
   - 嵌入式固件 → 需要硬件接口、外设、RTOS 章节
   - Web 应用 → 需要 API 路由、数据库、部署章节
   - CLI 工具 → 需要命令行参数、配置文件章节
   - 库/SDK → 需要集成指南、API 概览章节
   - 桌面应用 → 需要 UI 架构、状态管理章节

4. **提取业务逻辑**
   - 不仅要知道"有什么函数"，还要理解"为什么这样设计"
   - 识别核心算法和业务公式
   - 识别关键数据流
   - 识别常见故障点

### Phase 2: 结构设计

根据项目类型，从以下章节模板中选择合适的组合：

**通用章节（所有项目必选）：**
- 概述：项目定位、核心功能、技术栈
- 系统架构：分层架构图 + 目录结构
- 核心模块：关键模块的功能说明

**按项目类型选配：**

| 章节 | 嵌入式 | Web应用 | CLI工具 | 库/SDK |
|------|--------|---------|---------|--------|
| 硬件接口 | ✓ | | | |
| 外设驱动 | ✓ | | | |
| RTOS 任务 | ✓ | | | |
| API 路由 | | ✓ | | |
| 数据库 | | ✓ | | |
| 部署 | | ✓ | ✓ | |
| 命令行参数 | | | ✓ | |
| 配置文件 | ✓ | ✓ | ✓ | ✓ |
| 集成指南 | | | | ✓ |
| 核心算法 | ✓ | ✓ | ✓ | ✓ |
| 指令/协议 | ✓ | ✓ | | |
| FAQ/排障 | ✓ | ✓ | ✓ | ✓ |

### Phase 3: 内容生成

**使用 `template-guide.html` 作为 HTML 骨架。**

1. **复制模板**
   ```bash
   # 从 skill 目录复制模板到项目
   cp ~/.claude/skills/guide-generator/template-guide.html ./guide.html
   ```

2. **替换占位符**

   模板中的占位符：
   - `{{PROJECT_NAME}}` — 项目名称
   - `{{PROJECT_DESC}}` — 一句话描述
   - `{{PROJECT_META}}` — 技术栈元信息
   - `{{NAV_LINKS}}` — 导航栏链接
   - `{{SECTIONS}}` — 所有章节内容

3. **生成章节内容**

   每个章节遵循以下格式：
   ```html
   <section class="section" id="xxx">
     <h2 class="section-title">
       <span class="icon" style="background:...;color:...">图标</span>
       章节标题
     </h2>
     <p class="section-desc">一句话说明本章内容</p>
     <!-- 具体内容 -->
   </section>
   ```

4. **图表规范**

   **架构图**：使用 `<div class="diagram">` + ASCII art
   ```
   ┌─────────────────────────┐
   │      Layer Name         │
   ├─────────────────────────┤
   │  ┌─────┐    ┌─────┐    │
   │  │ Mod │───▶│ Mod │    │
   │  └─────┘    └─────┘    │
   └─────────────────────────┘
   ```

   **流程图**：使用 `<div class="flow">` + `<div class="flow-step">`
   ```html
   <div class="flow">
     <div class="flow-step"><b>Step 1</b><span>描述</span></div>
     <span class="flow-arrow">→</span>
     <div class="flow-step"><b>Step 2</b><span>描述</span></div>
   </div>
   ```

5. **代码块规范**

   使用 `<div class="code-block"><code>` 包裹，支持语法高亮 class：
   - `.cmt` — 注释
   - `.kw` — 关键字
   - `.fn` — 函数名
   - `.str` — 字符串
   - `.num` — 数字

6. **Callout 规范**

   ```html
   <div class="callout info|warning|success|danger">
     <span class="callout-icon">图标</span>
     <div>内容</div>
   </div>
   ```

7. **表格规范**

   使用标准 `<table>` + `<thead>` + `<tbody>`，模板 CSS 已内置样式。

8. **FAQ 使用折叠面板**

   ```html
   <details>
     <summary>问题标题</summary>
     <div>回答内容</div>
   </details>
   ```

### Phase 4: 质量校验

生成后必须检查：

| 检查项 | 标准 |
|--------|------|
| 架构图 | 必须有，ASCII 格式，层级清晰 |
| 代码引用 | 引用具体文件路径和函数名，不能泛泛而谈 |
| 业务逻辑 | 必须解释"为什么"，不能只说"是什么" |
| FAQ | 至少 3 个，覆盖常见故障 |
| 导航 | 所有 section id 必须与 nav-link href 对应 |
| 占位符 | 不能有未替换的 `{{xxx}}` |
| 响应式 | 检查移动端布局（card-grid, flow, table） |

## 内容质量要求

**必须做到：**
- 引用具体的文件路径、函数名、变量名、宏定义
- 解释核心算法的数学原理（如 OD 公式、定标算法）
- 列出关键的编译配置宏及其作用
- 提供可复制粘贴的代码片段
- FAQ 基于真实故障场景

**禁止：**
- 不读源码就猜测功能
- 使用"详见代码"等敷衍描述
- 跳过架构图
- 生成空章节或占位内容
- 照搬 README 内容（README 是入口，guide 是深度指导）

## 输出

- 文件名：`guide.html`
- 位置：项目根目录
- 格式：单文件 HTML，零外部依赖（CSS 内嵌）
- 特性：暗色/浅色主题自适应、响应式布局、粘性导航、打印优化

## 示例触发

```
> 生成指导文档
> 给这个项目写个guide
> generate project guide
> 帮我写个项目上手指南
```
