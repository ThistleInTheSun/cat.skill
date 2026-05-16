---
name: cat-create-skill
description: 引导用户在本仓库中创建一个新的 skill，遵循 cat-** 命名约定与目录结构。当用户提出"新建 skill / 创建 skill / 加一个 skill / author a new skill"等意图时使用。
---

# cat-create-skill

在 `cat.skill` 仓库中创建一个新的 skill。

不审阅已有 skill，不检测多个 skill 的功能重叠；这些场景分别交给 `cat-review-skill` 和 `cat-overlap-skills`。

## 仓库约定

- 顶层目录是**工作大类**，命名为 `cat-**`（例如 `cat-write` 写作类、`cat-skill` skill 元工作类）
- 每个具体 skill 是大类下的子目录，目录内必须包含 `SKILL.md`
- `SKILL.md` 顶部使用 YAML frontmatter，字段：
  - `name`：小写字母/数字/连字符，最长 64 字符
  - `description`：第三人称，包含 WHAT（做什么）和 WHEN（何时触发）

## 创建流程

复制并跟踪以下清单：

```
- [ ] 1. 确认目标大类（已有 cat-** 还是需要新建）
- [ ] 2. 命名 skill（lowercase + hyphens，建议带大类前缀或语义前缀）
- [ ] 3. 写 description（第三人称，含 WHAT + WHEN + 触发词）
- [ ] 4. 创建目录 <category>/<skill-name>/SKILL.md
- [ ] 5. 填写主体内容（指令 / 模板 / 示例 / 反例）
- [ ] 6. 自检：是否 < 500 行、术语一致、引用文件均为一级
- [ ] 7. 调用 cat-review-skill 做一次合规检查，修复所有 FAIL 项
```

### Step 1 — 确认大类

询问用户该 skill 属于哪个大类，或基于上下文推断。若不存在则新建一个 `cat-**` 顶层目录。

### Step 2 — 命名

- 仅使用 lowercase letters、数字、连字符
- 名字本身要能体现具体动作或对象，避免 `helper`、`utils` 这类模糊词
- 例：`cat-write/zhihu-tech-ip`、`cat-skill/cat-create-skill`

### Step 3 — 写 description

模板：

```
<动词短语描述能力>。当用户<触发场景 1>、<触发场景 2>，或提到<关键词列表>时使用。
```

反例：
- "帮助处理文档" — 太模糊
- "I can help you ..." — 第一人称
- "You can use this to ..." — 第二人称

### Step 4 — 创建文件

目录结构最小形态：

```
<category>/<skill-name>/
└── SKILL.md
```

按需扩展（progressive disclosure）：

```
<category>/<skill-name>/
├── SKILL.md
├── reference.md      # 详尽参考
├── examples.md       # 用例集
└── scripts/          # 可执行脚本
```

### Step 5 — 主体内容

按任务的"自由度"选择写法：

| 自由度 | 适用场景 | 写法 |
|--------|----------|------|
| 高 | 多种合理做法 | 自然语言指南 |
| 中 | 偏好一种但可变 | 模板 / 伪代码 |
| 低 | 易错、要求一致 | 具体脚本 / 严格步骤 |

### Step 6 — 自检

- [ ] SKILL.md 主体 < 500 行
- [ ] description 第三人称且含触发词
- [ ] 引用文件仅一级深度（不嵌套 `a.md` → `b.md` → `c.md`）
- [ ] 无 Windows 风格路径（路径分隔符使用正斜杠）
- [ ] 无时间敏感措辞（"截至 2025 年..."）

### Step 7 — 合规检查

完成后调用 `cat-skill/cat-review-skill` 对新 skill 做一次合规检查（PASS/FAIL），按报告修复所有 FAIL 项后再次检查，直至全部 PASS。

注意：review 只看"是否合规"，不评估能力质量；能力优化等未来的迭代 skill 上线后再做。

## 验证本 skill 是否起效

执行后产物应满足：

- 新 skill 目录和 `SKILL.md` 已按目标大类创建
- frontmatter 包含合规的 `name` 与 `description`
- 主体包含清晰的输入、工作流、输出或验证标准
- 经 `cat-review-skill` 检查后无 FAIL 项

## 反例

避免：
- 在 `cat-skill/` 下创建非 `cat-` 前缀的大类目录
- 把多个无关 skill 塞进同一个目录
- 在 SKILL.md 里堆砌 agent 已知的常识
