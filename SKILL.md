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
   cp ~/.claude/skills/guide-generator/template-guide.html ./guide.html
   ```

2. **替换占位符**

   模板中的占位符：
   - `{{PROJECT_NAME}}` — 项目名称（出现在 Hero、Topbar、Footer）
   - `{{PROJECT_DESC}}` — 一句话描述
   - `{{PROJECT_META}}` — 技术栈元信息
   - `{{SECTIONS}}` — 所有章节内容

   **注意**：侧边栏 TOC 由 JS 从 H2/H3 标题自动生成，无需手动维护导航链接。

3. **生成章节内容**

   **平面章节**（默认，适合大多数章节）：
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

   **可折叠章节**（适合参考性内容、附录、FAQ）：
   ```html
   <details class="section-block" id="xxx">
     <summary><h2>章节标题</h2></summary>
     <div class="section-body">
       <!-- 具体内容 -->
     </div>
   </details>
   ```

4. **组件规范**

   **架构图** — `<div class="diagram">` + ASCII art：
   ```html
   <div class="diagram">┌─────────────────────────┐
   │      Layer Name         │
   ├─────────────────────────┤
   │  ┌─────┐    ┌─────┐    │
   │  │ Mod │───▶│ Mod │    │
   │  └─────┘    └─────┘    │
   └─────────────────────────┘</div>
   ```

   **流程图** — `<div class="flow">` + `<div class="flow-step">`：
   ```html
   <div class="flow">
     <div class="flow-step"><b>Step 1</b><span>描述</span></div>
     <span class="flow-arrow">→</span>
     <div class="flow-step"><b>Step 2</b><span>描述</span></div>
   </div>
   ```

   **代码块** — 支持 highlight.js 自动高亮 + 手动 span 备用：
   ```html
   <!-- 推荐：language class 触发 highlight.js 自动高亮 -->
   <div class="code-block"><pre><code class="language-cpp">
   void setup() { pinMode(LED, OUTPUT); }
   </code></pre></div>

   <!-- 备用：手动 span（highlight.js 不支持的语言） -->
   <div class="code-block"><pre><code>
   <span class="cmt">// 注释</span>
   <span class="kw">auto</span> result = <span class="fn">compute</span>(<span class="num">42</span>);
   </code></pre></div>
   ```

   **Callout** — 5 种语义变体：
   ```html
   <div class="callout info|success|warning|danger|note">
     <span class="callout-icon">图标</span>
     <div><strong>标题</strong>内容</div>
   </div>
   ```

   **表格** — 带水平滚动包装：
   ```html
   <div class="table-wrapper">
     <table><thead><tr><th>...</th></tr></thead>
     <tbody><tr><td>...</td></tr></tbody></table>
   </div>
   ```

   **卡片网格** — 支持列数和颜色变体：
   ```html
   <div class="card-grid cols-2|cols-3">
     <div class="card accent-green|accent-amber|accent-red|accent-violet|accent-cyan|accent-blue">
       <h4>标题</h4><p>内容</p>
     </div>
   </div>
   ```

   **Tabs 标签页**（纯 CSS，最多 5 个）：
   ```html
   <div class="tabs">
     <input type="radio" name="t1" id="t1-1" checked>
     <input type="radio" name="t1" id="t1-2">
     <div class="tab-bar">
       <label for="t1-1">Tab 1</label>
       <label for="t1-2">Tab 2</label>
     </div>
     <div class="tab-panel">内容 1</div>
     <div class="tab-panel">内容 2</div>
   </div>
   ```

   **分层架构图**（Layer Stack）：
   ```html
   <div class="layer-stack">
     <div class="layer app">应用层 <span class="layer-label">描述</span></div>
     <div class="layer core">核心层 <span class="layer-label">描述</span></div>
     <div class="layer infra">基础设施 <span class="layer-label">描述</span></div>
     <div class="layer hw">硬件层 <span class="layer-label">描述</span></div>
   </div>
   ```

   **步骤指示器**（Step Indicators）：
   ```html
   <div class="steps">
     <div class="step" data-step="1"><h4>步骤名</h4><p>描述</p></div>
     <div class="step" data-step="2"><h4>步骤名</h4><p>描述</p></div>
   </div>
   ```

   **前置条件框**（Prerequisite）：
   ```html
   <div class="prereq">
     <h4>前置条件</h4>
     <ul><li>条件 1</li><li>条件 2</li></ul>
   </div>
   ```

   **依赖树**（Dependency Tree）：
   ```html
   <div class="dep-tree">ModuleName
    ├── Dep1
    ├── Dep2
    └── Dep3</div>
   ```

   **FAQ 折叠面板**：
   ```html
   <details class="faq">
     <summary>问题标题</summary>
     <div>回答内容</div>
   </details>
   ```

   **特性列表**（Feature List）：
   ```html
   <div class="feature-list">
     <div class="feature-item"><span class="check">✓</span> 特性描述</div>
   </div>
   ```

### Phase 4: 质量校验

生成后必须检查：

| 检查项 | 标准 |
|--------|------|
| 架构图 | 必须有，ASCII 格式，层级清晰 |
| 代码引用 | 引用具体文件路径和函数名，不能泛泛而谈 |
| 业务逻辑 | 必须解释"为什么"，不能只说"是什么" |
| FAQ | 至少 3 个，覆盖常见故障 |
| 占位符 | 不能有未替换的 `{{xxx}}` |
| 代码块 | 使用 `language-xxx` class 触发 highlight.js 高亮 |
| H3 id | 为 H3 标题添加 id 属性，确保 TOC 生成正确 |
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
- 格式：单文件 HTML，CSS 全部内嵌
- CDN 依赖：highlight.js（代码高亮，离线时回退到手动 span）
- 特性：
  - 侧边栏 TOC（H2/H3 自动生成）+ ScrollSpy
  - 顶部 Topbar（主题切换、移动端菜单）
  - 暗色/浅色主题切换（localStorage 持久化 + OS 偏好自动检测）
  - 代码块自动高亮 + 复制按钮
  - 响应式布局（900px 断点，移动端侧边栏抽屉）
  - 打印优化（隐藏导航、展开折叠、显示链接 URL）
  - ASCII 图表复制按钮

## 示例触发

```
> 生成指导文档
> 给这个项目写个guide
> generate project guide
> 帮我写个项目上手指南
```
