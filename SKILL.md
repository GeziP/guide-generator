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

4. **提取数据流（关键步骤，不可跳过）**
   - 追踪数据从输入到输出的完整路径：配置文件 → 解析 → 中间表示 → 执行 → 输出
   - 识别每个阶段的**具体数据结构**（字段名、类型、示例值）
   - 画出数据流图：哪些模块接收什么数据、输出什么数据
   - 找到一个具体的端到端场景（如：一个完整的配置文件如何被处理成最终执行）

5. **提取业务逻辑**
   - 不仅要知道"有什么函数"，还要理解"为什么这样设计"
   - 识别核心算法和业务公式
   - 识别常见故障点

### Phase 2: 结构设计（叙事式，非罗列式）

**核心原则：沿数据流组织章节，不是按组件分类罗列。**

Guide 是教程，不是参考手册。读者需要理解"数据怎么流过系统"，而不是"系统有哪些模块"。

**推荐叙事结构**（可根据项目调整）：

```
1. 概述 — 解决什么问题 + 整体架构（用数据流箭头的架构图，不是静态组件图）
2. 配置层 — 输入数据的结构（用具体 JSON/配置举例，逐层解释）
3. 处理管线 — 数据如何被转换（展开、冲突检测、依赖解析）
4. 执行引擎 — 数据如何驱动运行（调度循环、任务分发）
5. 任务执行 — 具体执行细节（接口、参数、策略）
6. 辅助系统 — 信号、错误处理等横切关注点
7. 完整示例 — 端到端场景（具体配置 → 运行流程 → 输出结果）
8. FAQ — 常见问题
```

**关键约束**：
- 第 7 章"完整示例"是**必须的**，不可省略
- 每个主要章节必须用**具体数据**举例（不能只列字段名）
- 架构图必须展示**数据流方向**（箭头标注数据类型），不是静态组件关系
- section 用 badge 标注类型：`Config`（配置相关）、`Runtime`（运行时相关）、`Example`（示例）

### Phase 3: 内容生成

**使用 `template-guide.html` 作为 HTML 骨架。**

**写作原则：教程式叙事，不是参考手册式罗列。**

- 每个章节从"这个阶段做什么"开始，然后用具体数据展示
- 代码嵌在解释段落中（"下面是 X 的定义，注意 Y 字段..."），不是孤立 code block
- callout 解释**具体机制**（"冲突判定：共享资源 → 同周期 → 时间重叠 → 不同模块"），不是泛泛而谈（"注意资源冲突"）
- 结构体/数据定义用**具体示例值**（`"module": "tongs", "defaultStart": 4000`），不是空表格列字段名

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
     <div class="section-header">
       <h2>章节标题</h2>
       <span class="section-badge badge-config|badge-runtime|badge-example">标签</span>
     </div>
     <div class="section-body">
       <!-- 具体内容 -->
     </div>
   </section>
   ```

   **Badge 类型**：
   - `badge-config` — 配置相关章节（蓝色）
   - `badge-runtime` — 运行时相关章节（绿色）
   - `badge-example` — 示例章节（琥珀色）

   **可折叠章节**（适合参考性内容、附录、FAQ）：
   ```html
   <details class="section-block" id="xxx">
     <summary><h2>章节标题</h2></summary>
     <div class="section-body">
       <!-- 具体内容 -->
     </div>
   </details>
   ```

   **完整示例章节**（必须，不可省略）：
   ```html
   <section class="section" id="sec-example">
     <div class="section-header">
       <h2>N. 完整示例：具体场景名</h2>
       <span class="section-badge badge-example">Example</span>
     </div>
     <div class="section-body">
       <h3>场景设定</h3>
       <!-- 描述一个具体的使用场景 -->

       <h3>配置文件</h3>
       <!-- 完整的配置文件（JSON/YAML/...），可运行 -->

       <h3>运行时流程</h3>
       <!-- 用 flow-diagram 展示数据在系统中的流转过程 -->

       <h3>输出结果</h3>
       <!-- 展开后的时序表 / 执行结果 / 输出数据 -->
     </div>
   </section>
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

### Phase 4: 质量校验（闭环）

生成后必须逐项检查。**任何一项不通过都必须修复后才能交付。**

**结构校验**：

| 检查项 | 标准 | 修复方式 |
|--------|------|----------|
| 占位符 | 不能有未替换的 `{{xxx}}` | 全文搜索替换 |
| 代码块 | 使用 `language-xxx` class 触发 highlight.js 高亮 | 添加 language class |
| H3 id | 为 H3 标题添加 id 属性，确保 TOC 生成正确 | 添加 id |
| 响应式 | 检查移动端布局（card-grid, flow, table） | CSS 调整 |
| badge | 每个 section-header 必须有 section-badge | 添加 badge |

**内容质量校验**（核心）：

| 检查项 | 标准 | 修复方式 |
|--------|------|----------|
| **完整示例** | 必须有"完整示例"章节，包含具体场景 + 完整配置 + 运行流程 + 输出表 | 补写示例章节 |
| **具体数据** | 每个主要组件必须用具体数据举例（不能只列字段名表格） | 用实际数据重写 |
| **架构图** | 必须展示数据流方向（箭头标注数据类型），不是静态组件框 | 重画架构图 |
| **代码嵌入** | 代码嵌在叙事段落中，不是孤立 code block | 调整代码位置 |
| **callout 精度** | callout 必须解释具体机制，不能泛泛而谈 | 重写 callout 内容 |
| **叙事连贯** | 章节之间有逻辑递进（上一章的输出是下一章的输入） | 添加过渡语句 |
| **FAQ** | 至少 3 个，覆盖常见故障场景 | 补充 FAQ |

**闭环验证**：校验完成后，重新通读全文，确认：
1. 一个新读者能否从头到尾理解数据如何流过系统？
2. 完整示例章节是否可独立理解（不依赖其他章节的知识）？
3. 是否有任何章节只是在"罗列"而非"讲解"？

## 内容质量要求

**必须做到：**
- 引用具体的文件路径、函数名、变量名、宏定义
- 用**具体数据**举例（JSON 片段带实际值，不是空模板）
- 代码嵌在叙事段落中，读者边读边看数据结构
- 架构图展示数据流方向（箭头 + 数据类型标注）
- callout 解释**具体判定条件/机制**，不是泛泛提醒
- 必须有"完整示例"章节（端到端：配置→流程→输出）
- FAQ 基于真实故障场景

**禁止：**
- 不读源码就猜测功能
- 使用"详见代码"等敷衍描述
- 跳过架构图
- 生成空章节或占位内容
- 照搬 README 内容（README 是入口，guide 是深度指导）
- 纯罗列式内容（只有表格列字段名，没有具体数据和解释）
- 孤立 code block（代码与文字脱节）

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
