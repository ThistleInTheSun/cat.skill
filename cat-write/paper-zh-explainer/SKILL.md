# paper-zh-explainer

## 触发条件
用户提供一个 PDF 文件路径，要求解读论文时使用。

---

## 必读资产

Step 2–4 执行前必须读取并遵守（与本 SKILL.md 同目录）：

- `voice-profile-blog.md`

其中句式 DO/DON'T、词汇白/黑名单、模式分档、自检清单，**全部当作硬约束**。

---

## 工作流

### Step 0 · 准备

从 PDF 文件名（不含扩展名）提取论文名称，记为 `PAPER_NAME`。
创建输出目录：

```
~/NeuralVault/paper_explainer/PAPER_NAME/
```

如目录已存在则继续。

---

### Step 1 · PDF 转 Markdown

运行：

```bash
markitdown <用户提供的PDF路径> -o ~/NeuralVault/paper_explainer/PAPER_NAME/pdf2txt_PAPER_NAME.md
```

若命令不存在，停止并提示用户：

```
markitdown 未安装，请运行：pip install markitdown
```

---

### Step 2 · 按 Section 分段，分配翻译 Agent

读取 `voice-profile-blog.md`，本步全文按**模式 A**（概念翻译 / 流程说明）执行其 DO/DON'T 与词汇白/黑名单。

读取 `pdf2txt_PAPER_NAME.md`，按论文 section 标题切分。

分配规则：
- Section 数 ≤ 10：每个 agent 负责一个 section；
- Section 数 > 10：将相邻小节合并，确保不超过 10 个 agent；
- 每个 agent 只读入分配给自己的行范围（起始行～结束行），不读入多余内容。

每个 Agent 的硬性要求：
- 翻译语言：中文大白话，口语化，不用学术腔；符合 voice profile 模式 A 的句式与词汇约束；
- 专有名词首次出现时在括号内加简短解释，例如：注意力机制（让模型聚焦关键信息的方法）；
- 直接翻译原文，不补充推断或背景；
- 公式用 Markdown 格式保留（`$...$` 或 `$$...$$`）；
- 插图和表格位置用 `【Figure X】` 或 `【Table X】` 占位，不描述图片内容；
- 翻译结果写入：`~/NeuralVault/paper_explainer/PAPER_NAME/sub_agent_XX.md`（XX 为两位数编号，如 01、02）。
- 不要出现分割线 `---`。

---

### Step 3 · 提取论文元信息卡

读取 `voice-profile-blog.md`，字段表述按**模式 A**（平实陈述，不端着）。

汇总所有 `sub_agent_XX.md`，从中提取以下字段：

```markdown
- 论文原名：
- 作者：（挑 1-3 位代表作者，注明机构；若无法确认履历，只写姓名）
- 团队 / 机构中文名：（使用中文常用名）
- 发表时间：
- 代码地址：
```

硬约束：
- 所有字段必须来自论文原文或用户提供材料；
- 无法确认的字段写"论文中未明确给出"，不编造；
- 不罗列全部作者，不硬编"大神"背景；
- 不得出现 voice profile 词汇黑名单中的词（赋能 / 加持 / 颠覆 / 综上所述 / ...）。

---

### Step 4 · 生成论文解读

读取 `voice-profile-blog.md`。主体按**模式 A**撰写；第 4 点「启发」可用**模式 B**（带情绪、正能量收束）。

按以下顺序撰写，大白话风格：

1. **领域与问题**：这篇论文属于什么领域，要解决什么问题；
2. **解决方法**：作者怎么解决这个问题；如有必要，指出最关键的一张图（`【Figure X】`）并用大白话解释它在说什么；
3. **实验验证**：用了什么实验方法，有哪些亮眼结果；
4. **启发**：这个方法给我们的启发是什么；

输出前跑 voice profile 的「全模式必过」+「模式 A 主导的文章」自检；「启发」段额外核对模式 B 的 DO 是否到位。论文解读部分不要出现分割线。过程中解读 1-2 个最重要的专有名词，格式为"名词（解释）"。

---

### Step 5 · 生成即梦配图 Prompt

根据论文标题自动填入 `[名称功能]`，输出固定风格 prompt：

```
艺术字设计，"[名称功能]"几个大字，字体为手写风格，字体为无衬线体，粗体，笔画粗细统一，部分笔画进行连笔处理，线条流畅灵动，有飘逸感，字体颜色为白色，黄色装饰。英文"for freedom"在文字旁，颜色为黄色，背景为深色，有些隐约的纹理，整体风格潮流，活力，具有运动氛围感。
```

---

### Step 6 · 合并输出最终文稿

将所有内容按以下顺序合并，写入：

```
~/NeuralVault/paper_explainer/PAPER_NAME/PAPER_NAME_解读.md
```

输出结构：

```markdown
## 配图 Prompt
（Step 5 内容）

---

## 论文元信息卡
（Step 3 内容）

---

## 论文解读
（Step 4 内容）

---

## 逐段大白话翻译
（所有 sub_agent_XX.md 内容按顺序拼接）
```
