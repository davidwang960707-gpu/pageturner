---
name: pageturner
description: "PageTurner Skill / 翻书人 Skill — 你的私人文档翻译+精排版专家。PDF / 网页 → 提取全文 → 结构化解析 → 中文翻译 → Apple 极简风 HTML 快速阅读页面。适用于研究报告、白皮书、长文、技术文档。"
---

# PageTurner Skill / 翻书人 Skill

把厚重的原文「翻」成可读的网页——你的私人文档翻译+精排版专家。

PDF / 网页 → 提取全文 → 结构化解析 → 中文翻译 → Apple 极简风 HTML。

## 加载时机

用户说「翻译这篇」「翻译这个报告/文档/PDF」「做个中文HTML版」「快速阅读版」「翻书人」等关键词时加载。

## 工作流程（5 步）

### 步骤 1：获取原文

| 原文来源 | 工具 | 方法 |
|---------|------|------|
| **PDF 文件** | `terminal: python3 -c "import pdfplumber; ..."` | 用 pdfplumber 逐页 `extract_text()`，输出到 `/tmp/<name>_text.txt` |
| **网页** | `browser_navigate` 或 `terminal: curl` | 优先 curl（快），动态页面用 browser；避免搜索引擎（CAPTCHA 概率高） |
| **本地 .txt/.md** | `read_file` | 直接读 |

**PDF 提取代码模板**：
```python
import pdfplumber
with pdfplumber.open("/path/to/doc.pdf") as pdf:
    all_text = []
    for i, page in enumerate(pdf.pages):
        text = page.extract_text()
        if text:
            all_text.append(f"--- PAGE {i+1} ---\n{text}")
    full_text = "\n\n".join(all_text)
    with open("/tmp/doc_text.txt", "w") as f:
        f.write(full_text)
```

**搜索引擎 CAPTCHA 经验**：
- Google → 大概率触发 bot 检测
- DuckDuckGo → 会出 CAPTCHA 挑战
- Bing → 部分情况要验证
- **优先策略**：直接访问发布机构官网搜索，而非搜索引擎；已知机构则直接猜 URL（如 `/publication/<slug>/`）

### 步骤 2：结构解析

读取全文后，识别文档骨架：

| 需要识别 | 正则/方法 | HTML 对应 |
|---------|----------|----------|
| 封面信息 | 前几页的标题行 + 作者行 | `.cover` 区域 |
| 章节 | `Chapter \d+` / `第.*章` / 数字编号 | `.chapter-num` + `.chapter-title` |
| 子标题 | 全大写行 / `Finding \d+` / `###` | `h3` |
| 案例研究 | `CASE STUDY` / `案例` | `.case-study` 框 |
| 引用 | `"..."` 后跟 `— Name` | `blockquote` |
| 表格 | 连续的行含 `\t` 或对齐空格 | `<table>` |
| 数据亮点 | `XX%` `N→M` `$XM` | `.stat-bar` 或 `<table>` |
| 尾注/参考文献 | `[1]` `[2]` 或 `Endnotes` | 小字号列表 |

**关键**：解析时标注每段的「语义角色」（finding / quote / case / stat / body），翻译时保持角色不变。

### 步骤 2.5：数据看板提取（v2 新增）

在翻译前，扫描全文提取「关键数字」，用于生成封面后的数据速览看板：

**提取规则**：
| 数据类型 | 正则 | 看板展示 |
|---------|------|---------|
| 百分比对比 | `(\d+)%` 前后有 `vs` `from` `to` `→` | 大号数字 + 变化箭头 |
| 核心统计 | 样本量、覆盖范围（如 `51 cases` `41 organizations`） | 4-6 列 stat-bar |
| N→M 变化 | `(\d+)\s*→\s*(\d+)` | 前后对比条 |
| 金额 | `\$\d+[MBK]` | 高亮金额 |
| 排名/位置 | `top \d+` `#\d+` | 标签徽章 |

提取后生成 `.stats-dashboard` 区块，放在封面后、前言前。

### 步骤 3：翻译策略

**目标语言**：默认中文（zh-CN），用户可指定其他语言。

**翻译原则**：
1. **保持原文逻辑结构**——不合并段落、不拆分要点
2. **数据原样保留**——百分比、金额、数字不变
3. **专有名词保留英文**——公司名、产品名、人名不翻译（如「GitHub Copilot」不翻成「GitHub 副驾驶」）
4. **引用保持口语感**——高管原话用引号 + 第一人称
5. **术语统一**——同一英文术语全文用同一中文译法

**术语对照表（企业 AI 领域常用）**：
| 英文 | 中文 |
|------|------|
| deployment | 部署 / 落地 |
| implementation | 实施 |
| executive sponsor | 高管赞助者 / 支持者 |
| change management | 变革管理 |
| headcount reduction | 人力缩减 / 减员 |
| escalation model | 上报模式 |
| agentic AI | Agentic AI / 自治 AI |
| shadow AI | Shadow AI / 影子 AI |
| proof of concept | 概念验证 |
| ROI | ROI / 投资回报率 |
| OKR | OKR / 目标与关键结果 |
| SME (subject matter expert) | 主题专家 |

**规模控制**：
- < 20 页文档：全文翻译
- 20-50 页：全文翻译，案例研究精简
- 50+ 页：章节 Finding 全译，案例研究只保留关键数据和一句教训
- 100+ 页：核心发现 + Finding 全译，案例研究精简，附录保留 KPI 表

### 步骤 4：HTML 生成

**设计系统**：Apple WWDC 极简文档风格。完整 HTML/CSS/JS 骨架见 `templates/fastread.html`。

**模板读取协议**：
1. 先读取 Skill 根目录下的 `templates/fastread.html`，并以它作为 HTML/CSS/JS 骨架。
2. 在 Hermes / Claude 环境中，优先用 `skill_view(name="pageturner", file_path="templates/fastread.html")` 或等价的 Skill 资源读取能力加载模板；如果有文件系统访问权，再读取 `<skill_dir>/templates/fastread.html`。
3. 不要只因为当前工作目录找不到 `templates/fastread.html` 就判定模板不可用；相对路径必须以 Skill 根目录为基准。
4. 只有在该 Skill 是通过单文件 `SKILL.md` URL 安装、或运行环境没有同步/挂载 `templates/` 目录时，才允许说模板文件不可用；此时按下方设计规范重建同等结构，并在最终说明中提示需要用完整仓库安装 Skill。

**四类核心输出能力**：

#### 4a. 数据速览看板

封面后自动插入 `.stats-dashboard`，从步骤 2.5 提取的数据生成 4-6 列大号数字瓷砖（`.stat-bar` 样式）、含 N→M 变化的对比条、关键百分比高亮。

#### 4b. 键盘翻页导航（v2 新增）

每个 `<section>` 作为一页，支持：← → 键翻页、顶部 2px 进度条、右下角页码（如 `3 / 22`）、URL hash 同步（`#s3`）、Home/End 跳首末页。移动端额外支持左右滑动手势。详细 JS 见模板文件。

#### 4c. 原文链接（v2 新增）

封面区底部或页脚添加原文本地文件链接（`file://` 绝对路径）+ 官网 URL（如有）。样式：小字号 muted，底部边框分隔。

#### 4d. 图解与数据可视化（v2.1）

不要只输出摘要文字。只要原文含有架构、流程、分类、实验数据、治理框架或概念关系，必须提取成可视化模块。优先使用自包含 HTML/CSS 图表；如原文有真实图片且授权允许，可引用或嵌入真实图片。不要用 emoji、ASCII 画图或装饰性占位图。

**最低要求**：
- 20 页以上文档：除 `.stats-dashboard` 外，至少 2 个 `.figure-panel` / 图解模块。
- 论文类文档：至少包含 1 个结构图或流程图 + 1 个表格/柱状图/矩阵。
- 企业报告类文档：至少包含 1 个关键指标图 + 1 个案例/风险/路线图矩阵。
- 如果数字来自原文，保留原数；如果是解释性示意图，标题或说明中标注“示意”。

**推荐图解类型**：
| 文档类型 | 图解模块 | 适合内容 |
|---------|----------|----------|
| AI / LLM 论文 | architecture map / heatmap / complexity bars / training table | 模型结构、Q/K/V、实验指标、训练配方 |
| 本体论 / 知识图谱论文 | workflow / concept map / CQ list / risk matrix | 用例、能力问题、类关系、模式复用 |
| 企业报告 / 白皮书 | dashboard / case matrix / adoption funnel / roadmap | 关键数字、组织变化、案例分组、落地路径 |
| 方法论文 | process map / decision matrix / checklist | 步骤、判断条件、失败模式 |

**CSS 变量规范**：
```css
:root {
  --ink: #1d1d1f;         /* 正文 */
  --ink-muted: #6e6e73;   /* 次要文字 */
  --bg: #ffffff;          /* 主背景 */
  --bg-alt: #f5f5f7;      /* 交替背景 */
  --accent: #7b61ff;      /* 强调色（仅用于标签/分割线/左侧条） */
  --divider: rgba(0,0,0,0.08);
  --quote-bg: #fafafa;
  --case-bg: #f8f7ff;    /* 案例研究淡紫背景 */
  --case-border: rgba(123,97,255,0.2);
}
```

**布局节奏**：
```
封面（100vh，渐变背景，顶部 4px 彩色细线）
  ├── eyebrow 标签（13px，大写，紫色）
  ├── h1 主标题（42-80px，letter-spacing: 0）
  ├── subtitle 副标题（18-26px，muted）
  └── meta 作者/机构

章节区（section，800px 最大宽度居中，padding 80px）
  ├── 白底 / 浅灰底交替（section.alt）
  ├── chapter-num（13px 紫色小型大写）
  ├── chapter-title（32px 粗体）
  ├── chapter-sub（18px muted）
  ├── finding 块（finding-label + h3 + p）
  ├── blockquote（左侧 3px 紫条 + 浅灰背景 + 斜体）
  ├── case-study 框（淡紫背景 + 圆角 12px + 紫边框）
  │     ├── case-label（11px 紫色小型大写）
  │     ├── h4 案例标题
  │     └── case-meta（13px muted）
  ├── table（无竖线，斑马纹，tabular-nums）
  ├── stat-bar（flex 数字瓷砖）
  └── figure-panel（结构图 / 图表 / 矩阵）

附录区
  ├── KPI 对照表
  ├── 失败模式表
  └── 尾注（小字号）

页脚（居中，分割线，muted 文字）
```

**排版关键参数**：
- `font-size: 18px` 基础字号
- `line-height: 1.7-1.75` 正文行高
- `h2: 36px / h3: 24px / h4: 18px`
- 大标题 `letter-spacing: 0`
- 标签 `letter-spacing: 0.06em ~ 0.14em`
- 表头 `font-size: 12px, letter-spacing: 0.06em`
- 响应式：768px 断点缩小间距和字号

**硬规则**：
- **零 emoji**（用 SVG 或纯文本符号）
- **零卡片容器**（不用 `border+radius+shadow+padding` 四件套）
- **强调色面积 < 5%**（仅标签、分割线、左侧条）
- **数字用 tabular-nums**（`font-variant-numeric: tabular-nums`）
- **中文默认**（i18n 不依赖 navigator.language）

### 步骤 5：输出与验证

**输出路径**：`/Users/<user>/Desktop/<Document-Name>-zh.html`

**验证清单**：
- [ ] 浏览器打开 HTML，检查封面渲染
- [ ] 滚动检查章节结构完整
- [ ] 检查所有表格正常显示
- [ ] 检查引用块格式
- [ ] 检查案例研究框样式
- [ ] 20 页以上或论文类输出至少有 2 个图解/数据可视化模块
- [ ] 检查移动端响应式（768px 断点）
- [ ] `document.body.innerText.length` > 原文 40%+（翻译充分）
- [ ] 无 JavaScript 报错（console 检查）

## 大文件处理技巧

对于 50+ 页文档，**不要一次性在 response 中生成全部 HTML**：

1. **先写框架 + 前半部分**（用 `write_file`）
2. **用 `patch` 追加后半部分**（old_string 锚定 `</body>\n</html>`）

```python
# patch 追加模式
patch(
    path="output.html",
    old_string="</body>\n</html>",
    new_string="<!-- 新内容 -->\n</body>\n</html>"
)
```

## 已踩过的坑

| 坑 | 症状 | 解法 |
|----|------|------|
| 搜索引擎 CAPTCHA | curl/browser 返回验证页面 | 直接访问机构网站，用站内搜索，或猜 URL |
| PDF 太大一次性写不下 | write_file 内容过长 | 分 2-3 次 patch 追加 |
| vision API 不可用 | 无法截图验证 | 用 `browser_console` 执行 JS 统计数据 |
| macOS grep 无 -P | grep 正则报错 | 用 Python `re.findall` 代替 |
| pdfplumber 在 sandbox 不可用 | execute_code 报 ModuleNotFoundError | 用 terminal 直接跑 python3 |

## 输出物规格

| 属性 | 值 |
|------|-----|
| 输出格式 | 自包含单文件 HTML（CSS 内联） |
| 文件大小 | 通常 50-150KB（100+ 页原文） |
| 语言 | 中文（可配置） |
| 设计风格 | Apple WWDC 极简文档风 |
| 字体 | PingFang SC / Noto Sans SC / SF Pro Display |
| 响应式 | 是（768px 断点） |
| 依赖 | 零外部依赖（纯 HTML+CSS） |
| 可打印 | 是（`@media print` 可加） |

## 参考产出

`/Users/macbook/Desktop/Enterprise-AI-Playbook-zh.html` — 斯坦福《企业 AI 实战手册》完整中文翻译版（116 页原文 → 70KB HTML，11 章 + 11 案例 + 附录）。
