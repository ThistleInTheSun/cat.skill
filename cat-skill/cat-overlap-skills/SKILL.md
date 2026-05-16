---
name: cat-overlap-skills
description: 扫描指定目录下的多个 skill，提取能力指纹并两两比对，检测功能重叠、触发条件冲突和 description 近义，输出去重/边界化行动项；不做单个 skill 基础合规检查或能力改进评测。当用户提出检查 skill 是否重复、skill 查重、检测 skill overlap、审计 skill 边界或优化多个 skill 边界时使用。
---

# cat-overlap-skills

对指定目录下的所有 skill 做两两比对，检测**功能重叠（overlap）**，输出可执行的去重或边界化建议。

> 与 `cat-review-skill` 的区别：那个对**单个** skill 做基础合规检查（PASS/FAIL）；本 skill 审多个 skill 之间的**重叠关系**。两者都不修改文件，落地交给用户或 `cat-create-skill`。

## 输入

- `target`：目录路径
  - 仓库根（如 `D:/cat.skill/`）→ 全量审计，按大类分组
  - 单个大类（如 `cat-write/`）→ 只审该大类内部
- 若用户未提供 `target`：
  - 当前工作区能明确判断仓库根 → 使用仓库根
  - 无法判断范围 → 先询问用户要审计的目录

## 工作流

复制并跟踪：

```
- [ ] 1. 收集 skill 清单（递归找 SKILL.md）
- [ ] 2. 提取每个 skill 的能力指纹
- [ ] 3. 归一化能力指纹，合并近义触发词
- [ ] 4. 两两比对，检测 overlap 与误报
- [ ] 5. 生成审计报告
- [ ] 6. 提出行动项（合并 / 拆分边界 / 重命名 / 改写 description）
```

### Step 1 — 收集

递归查找 `target` 下所有 `SKILL.md`。对每个 skill 记录：

- 路径 `<category>/<skill-name>/SKILL.md`
- frontmatter 的 `name` 与 `description`
- 主体的一级标题与"指令/工作流"小节要点

跳过：`SKILL.md` 缺失或 frontmatter 不合规的目录（标记为"结构问题"另列）。

### Step 2 — 提取能力指纹

为每个 skill 抽出三元组：

```
{
  "object":   <作用对象>,     例如：单个 skill、一组 skill、文章、PR
  "action":   <核心动作>,     例如：create、review、audit、publish、refactor
  "trigger":  <触发关键词集合> 来自 description 中"当用户...时使用"的词
}
```

同时记录用于降低误报的辅助字段：

```
{
  "scope":     <作用范围>,       例如：单个 skill、目录内多个 skill、全仓库
  "non_goals": <明确不做什么>,
  "workflow":  <关键步骤摘要>,
  "output":    <报告、diff 建议、完整文件草案等产物形态>
}
```

### Step 3 — 归一化指纹

比较前先做轻量归一化：

- 将 `review` / `审阅` / `检查` 归为同类动作，但保留对象差异
- 将 `improve` / `优化` / `提升表现` 归为同类动作，但区分单个 skill 与多个 skill
- 将 trigger 中的同义词合并后再计算交集
- 若两个 skill 明确互相点名并声明边界，标记为"分层关系"，不直接判为冲突

### Step 4 — 两两比对检测 overlap

对所有 skill 两两比较，按以下信号打分：

| 信号 | 权重 |
|------|------|
| `action` 相同且 `object` 高度重合 | 高 |
| `trigger` 关键词交集 ≥ 50% | 高 |
| `name` 词根相同（如都含 `review`） | 中 |
| 主体工作流步骤高度雷同 | 中 |
| description 表述近义但措辞不同 | 低 |

只要命中 ≥ 1 个"高"或 ≥ 2 个"中"，即列入 overlap 候选。

跨大类比较时仍要做：同动词、同对象的 skill 出现在不同大类是常见的边界错位。

候选进入报告前，先判断是否属于以下误报：

- **分层互补**：一个做粗筛，一个做深度改稿，且 description 或开篇已声明边界
- **同对象不同动作**：都处理 skill，但一个 review、一个 improve、一个 audit overlap
- **同动作不同对象**：都 review，但对象分别是单个 skill、PR、文章等

误报不列入 Overlap 主节；如有必要，在"边缘情况"中简短说明。

### Step 5 — 生成报告

按以下格式输出：

```markdown
# Audit: <target>

## 概览
- 扫描目录：<target>
- skill 总数：N
- 按大类分布：cat-skill (X), cat-write (Y), ...
- 结构问题：M 个（详见末尾）
- 检出 overlap 对数：K
- 边缘情况：B 个

## Overlap（功能重叠）
### O1: <skill-a> ↔ <skill-b>
- 命中信号：<动作相同 / 触发词重合 70% / ...>
- 严重度：<高 / 中 / 低>
- 证据：
  - <skill-a> description: "..."
  - <skill-b> description: "..."
- 判断：<为什么这是 overlap，而不是分层互补>
- 处置建议：<合并 / 拆分边界 / 重命名 / 重写 description 以消歧>

## 边缘情况（不计入 overlap）
- <skill-a> ↔ <skill-b>: <疑似重叠信号> -> <为什么暂不处理>

## 结构问题
- <路径>: <frontmatter 缺失 / name 与目录不一致 / ...>

## 行动项（按优先级）
1. <动作类型>: <目标路径> -> <关键 diff 片段或合并草稿>
2. <动作类型>: <目标路径> -> <关键 diff 片段或合并草稿>
```

### Step 6 — 行动项

每条行动项必须可直接执行，给出：

- 动作类型：`MERGE` / `SPLIT` / `RENAME` / `REWRITE_DESCRIPTION`
- 目标路径
- 关键 diff 片段（旧 description → 新 description；或合并后的 frontmatter 草稿）

## 审计原则

1. **重叠不一定坏**：若两个 skill 故意分层（粗粒度 + 细粒度），保留但必须在双方 description 中互相点名以明确边界
2. **优先看 description**：description 是 agent 选 skill 的依据，重叠的 description 比重叠的实现更危险
3. **结构问题单列**：frontmatter 错误、目录命名不规范等不算 overlap，单列在"结构问题"节
4. **不修改文件**：本 skill 只产出报告与建议，落地改动由用户或 `cat-create-skill` 执行（`cat-review-skill` 同样只输出报告，不做修改）

## 验证本 skill 是否起效

执行后产物应满足：

- 报告中明确给出扫描目录、skill 总数、结构问题数量和 overlap 对数
- 每个 overlap 候选都列出命中信号与证据
- 每条行动项都包含动作类型、目标路径和可执行建议
- 结构问题与 overlap 问题分开展示

## 反例

- ❌ 仅看名字判定重叠（`*-skill` 后缀会造成大量假阳性）
- ❌ 列出所有差异点而不区分严重程度
- ❌ 报告里只说"建议合并"却不指出合并后的目标 description
- ❌ 把"结构问题"（如 frontmatter 错误）和"重叠"混在同一节
- ❌ 跨大类的 overlap 因为"分属不同大类"就放过
