# PageTurner Skills / 翻书人 Skills 

<div align="center">

  <img src="https://github.com/davidwang960707-gpu.png" width="120" height="120" alt="王六 avatar" />

  <h3>PageTurner · PDF / 长文档翻译与 HTML 快读版生成 Skill</h3>

  <h3>把厚重文档翻成能读完的网页</h3>

  <p>
    把 PDF、网页、论文和研究报告，从“能打开”推进到“能读完、能分享、能发布”：提取全文、理解结构、翻译成中文，再生成一个自包含的精排版 HTML 快读页面。
  </p>

  <p>
    <img src="https://img.shields.io/badge/Document-Translation%20%26%20Layout-111827?style=for-the-badge&logo=googledocs&logoColor=white" alt="Document Translation and Layout" />
    <img src="https://img.shields.io/badge/Fast%20Read-HTML%20Pages-7B61FF?style=for-the-badge&logo=html5&logoColor=white" alt="Fast Read HTML Pages" />
    <img src="https://img.shields.io/badge/GitHub%20Pages-Publish%20Ready-0EA5E9?style=for-the-badge&logo=githubpages&logoColor=white" alt="GitHub Pages Publish Ready" />
  </p>

  <p>
    <a href="https://davidwang960707-gpu.github.io/pageturner/">在线预览：PageTurner 示例站</a>
  </p>

  <p>
    <img src="https://img.shields.io/badge/Hermes%20%2F%20Claude-Skill-4F46E5?style=for-the-badge&logo=anthropic&logoColor=white" alt="Hermes Claude Skill" />
    <img src="https://img.shields.io/badge/Zero%20Dependency-Single%20HTML-22C55E?style=for-the-badge&logo=files&logoColor=white" alt="Zero Dependency Single HTML" />
    <img src="https://img.shields.io/badge/Data%20Viz-Readable%20Reports-F59E0B?style=for-the-badge&logo=chartdotjs&logoColor=white" alt="Data Visualization Readable Reports" />
  </p>
</div>

---

## 这是什么

PageTurner 不是“把 PDF 逐句丢进翻译器”的小脚本。它更像一个耐心的研究助理：

1. 先把 PDF / 网页 / 文本文档里的内容提取出来；
2. 识别标题、章节、发现、引用、案例、表格和关键数字；
3. 保留原文结构与数据，翻译成中文；
4. 用 `templates/fastread.html` 生成一个可本地打开、可放到 GitHub Pages 的 HTML 快读版。

适合处理研究报告、企业白皮书、AI 论文、知识图谱 / 本体论论文、长篇技术文档。目标很朴素：别让好文档死在 116 页 PDF 里。

---

## Status

| 模块 | 状态 | 说明 |
|---|---|---|
| Skill 主体 | ready | `SKILL.md` 已包含触发词、工作流程、翻译策略和验证清单 |
| HTML 模板 | ready | `templates/fastread.html` 是零依赖单文件模板，支持键盘翻页与移动端滑动 |
| GitHub Pages 展示 | ready | `docs/` 可直接作为 GitHub Pages 静态站发布 |
| 示例结果 | ready | `docs/examples/` 放置三类任务完成结果示例 |

---

## 能力边界

PageTurner 默认输出中文快读版，强调“可读、结构清楚、数字可靠”。它会尽量保持专有名词、数据、表格和引用的语义角色，但它不是法律认证翻译，也不是出版级校译流程。

建议把它用于：

- 快速读懂英文研究报告和白皮书；
- 把 AI / KG / 本体论论文转成中文阅读页；
- 给团队做资料预读、内部分享和研讨材料；
- 为 GitHub Pages、Notion、知识库准备轻量 HTML 读物。

不建议把未经人工复核的结果直接用于：

- 法律、医疗、财务等高风险决策；
- 对外发布的官方译本；
- 需要逐字一致性的学术引用场景。

---

## 快速开始

把仓库放到 Hermes / Claude 的 Skills 内容目录：

```bash
# 先进入你 clone 下来的 pageturner 仓库
cd pageturner

mkdir -p ~/.hermes/profiles/claude/skills/content
ln -s "$(pwd)" ~/.hermes/profiles/claude/skills/content/pageturner
```

如果不想用软链接，也可以直接把整个目录复制到 `~/.hermes/profiles/claude/skills/content/pageturner`。

然后在 Claude / Hermes 中这样叫它：

```text
用翻书人把这份 PDF 做成中文 HTML 快速阅读版：
https://example.com/report.pdf
```

也可以指定本地文件：

```text
翻译 /Users/me/Downloads/paper.pdf，输出一个适合 GitHub Pages 展示的中文快读版。
```

默认输出是一个自包含 HTML 文件，通常放在桌面或你指定的项目目录中。没有前端依赖，没有构建步骤，浏览器直接打开。

---

## 示例结果

这些示例适合放在 GitHub Pages 上作为项目展示页。完整静态站入口见 [`docs/index.html`](docs/index.html)。

| 示例 | 原文类型 | 输出结果 |
|---|---|---|
| Enterprise AI Playbook | 斯坦福企业 AI 报告 | [`docs/examples/enterprise-ai-playbook.html`](docs/examples/enterprise-ai-playbook.html) |
| Attention Is All You Need | AI 大模型经典论文 | [`docs/examples/attention-is-all-you-need.html`](docs/examples/attention-is-all-you-need.html) |
| Modular Ontology Modeling | 本体论 / 知识建模论文 | [`docs/examples/modular-ontology-modeling.html`](docs/examples/modular-ontology-modeling.html) |

第一个示例来自斯坦福 Digital Economy Lab 的 [Enterprise AI Playbook](https://digitaleconomy.stanford.edu/app/uploads/2026/03/EnterpriseAIPlaybook_PereiraGraylinBrynjolfsson.pdf)。另外两个示例展示 AI 大模型论文、本体论论文如何被整理成“封面 + 数据速览 + 章节快读 + 参考链接”的阅读页形态。

示例页不只是摘要：AI 论文示例包含架构图、注意力热力图和复杂度图表；本体论示例包含流程图、概念图、能力问题清单和风险矩阵。

---

## GitHub Pages 发布

这个仓库已经按 GitHub Pages 的静态站习惯准备了 `docs/` 目录：

```text
pageturner/
├── SKILL.md
├── templates/
│   └── fastread.html
├── docs/
│   ├── index.html
│   ├── .nojekyll
│   ├── assets/
│   └── examples/
│       ├── enterprise-ai-playbook.html
│       ├── attention-is-all-you-need.html
│       └── modular-ontology-modeling.html
├── README.md
├── LICENSE
└── .gitignore
```

发布步骤：

1. 推到 GitHub；
2. 打开仓库 `Settings -> Pages`；
3. Source 选择 `Deploy from a branch`；
4. Branch 选择 `main`，目录选择 `/docs`；
5. 保存后等待 GitHub 生成访问地址。

之后你只需要把新的 PageTurner 输出 HTML 放进 `docs/examples/`，再在 `docs/index.html` 和本 README 的示例表里加一行。

---

## Skill 如何工作

核心流程写在 [`SKILL.md`](SKILL.md)：

1. 获取原文：PDF 用 `pdfplumber`，网页优先直连来源站；
2. 结构解析：识别封面、章节、Finding、案例、引用、表格和数据亮点；
3. 翻译策略：保持结构、保留数字、统一术语、专有名词不乱翻；
4. HTML 生成：套用 Apple 文档风格的 `fastread.html`；
5. 输出验证：浏览器打开、滚动检查、移动端检查、确认文本量和 JS 状态。

`templates/fastread.html` 提供了：

- 进度条和页码；
- 左右方向键翻页；
- Home / End 跳转；
- 移动端上下滑动；
- 数据速览看板；
- 表格、引用、案例研究、关键发现等样式。

---

## 质量原则

PageTurner 的审美很克制：少装饰，多留白；少炫技，多结构。翻译时遵守几条硬规则：

- 数字不改，单位不乱；
- 表格不拍扁，章节不揉碎；
- 公司名、产品名、人名保留英文；
- 术语前后一致；
- 读者能在 5 分钟内知道“这篇文档值不值得继续读”。

这也是为什么它输出 HTML，而不是一大段聊天记录。文档应该有边界、有节奏、有回看的入口。

---

## 贡献

欢迎提 issue / PR，尤其是这些方向：

- 更稳的 PDF 表格提取；
- 更多领域术语表；
- 更好的双语对照模式；
- 更多可发布的示例结果；
- 对不同 Skill 运行时的安装说明。

保持一个小原则：模板可以漂亮，但不要喧宾夺主。读者是来读文档的，不是来看模板表演的。

---

## License

MIT. 详见 [`LICENSE`](LICENSE)。
