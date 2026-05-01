# Research Workflow Skill

> 一套 AI 协同的实验管理工作流。两种模式：冷启动搭建 + 热更新运行。

---

## 模式判断

AI 读到这个文件后，第一步：检查 `research/` 目录是否存在且包含 `ROADMAP.md`。

- **不存在或不完整** → 执行「冷启动」
- **存在且完整** → 执行「热更新」

---

# Part 1: 冷启动

> 一次性执行。目标：从零搭建完整的实验管理基础设施。

## 1.1 创建目录结构

```
research/
├── BOOTSTRAP.md          # AI 启动手册
├── ROADMAP.md            # 论文 story + 证据链
├── HYPOTHESES.md         # 假设池
├── RESPONSIBILITY.md     # 职责分工
├── WRITING.md            # 写作进度
├── WEEKLY.md             # 回顾日志
├── scripts/
│   ├── new_exp.py        # 创建实验
│   └── status.py         # 扫描进度
└── experiments/
    └── _template.md      # 实验模板
```

## 1.2 写入文件

### BOOTSTRAP.md

```markdown
# Research Workflow — AI Onboarding

## 项目背景
<!-- 人填：项目名称、课题、核心主张 -->

## 启动流程（每次新 session 执行）
1. 读 research/ROADMAP.md → 了解全局状态和当前优先级
2. 读 research/HYPOTHESES.md → 了解假设池状态
3. 读最近 1-2 个实验文件 → 了解最新进展
4. 运行 python research/scripts/status.py → 查看整体进度
5. 告诉用户当前状态摘要，询问要做什么

## 职责分工
详见 research/RESPONSIBILITY.md。

## 实验闭环流程
详见本文件 Part 2: 热更新。

## 关键规则
- AI 绝不写结论。结论是研究判断，必须由人完成。
- 每个实验必须有 Question 和 Design，不能"跑跑看"。
- Next 里的 action items 要具体，不能写"进一步分析"。
- 结果数据由 AI 提取，但人必须验证一次。
- 新假设必须关联到具体实验，不能悬空。
```

### ROADMAP.md

```markdown
# Story
<!-- 人填：论文的核心主张，一句话 -->

# Evidence Chain
<!-- 人填：支撑 story 的证据链 -->
- [ ] 证据 1 → EXPXXX
- [ ] 证据 2 → EXPXXX

# Current Priority
<!-- 人填：当前最重要的 1-3 件事 -->

# Dead Ends
<!-- 记录走过的死路，避免重复 -->
```

### HYPOTHESES.md

```markdown
# 假设池

## 待验证
<!-- 新假设放这里，必须关联实验 -->

## 已验证
<!-- 实验确认成立的假设移到这里 -->

## 已推翻
<!-- 实验推翻的假设移到这里 -->

## 已放弃
<!-- 主动放弃的假设移到这里 -->
```

### RESPONSIBILITY.md

```markdown
# 职责分工

## AI 独立做
- [✓] 创建/编号实验文件（new_exp.py）
- [✓] 从 CSV/TSV 提取结果数据填入实验 Results
- [✓] 从配置文件提取实验参数填入 Config
- [✓] 更新假设状态（基于已确认的实验结论）
- [✓] 生成 status 报告
- [✓] 起草 Method / Experiments / Related Work 段落

## AI 起草，人必须审阅
- [◐] 实验设计方案（Design）
- [◐] 分析段落（Analysis — 模式发现）
- [◐] ROADMAP 更新
- [◐] 论文 section 初稿（Method / Experiments / Related Work）

## 人独立做，AI 不碰
- [✗] 定义论文 Story 和研究方向
- [✗] 提出假设
- [✗] 决定实验方向
- [✗] 写结论（Conclusion 段必须人写）
- [✗] 判断假设是否成立
- [✗] 决定下一步优先级
- [✗] 写 Introduction / Conclusion
```

### WRITING.md

```markdown
# 写作进度

## Section 状态
- [ ] Introduction: 未开始
- [ ] Related Work: 未开始
- [ ] Method: 未开始
- [ ] Experiments: 等实验结果
- [ ] Conclusion: 未开始

## 写作阻塞
<!-- 哪些 section 被什么实验阻塞了 -->
```

### WEEKLY.md

```markdown
# 回顾日志

## YYYY-MM-DD
### 本周做了什么
-

### 学到了什么
-

### 下周计划
-

### 当前最大风险
-
```

### experiments/_template.md

```markdown
---
id: {{{id}}}
status: active
hypotheses: [{{{hypothesis_ids}}}]
model: {{{model}}}
dataset: {{{dataset}}}
created: {{{date}}}
updated: {{{date}}}
---

## Question
<!-- 这个实验要回答什么问题？一句话。人写。-->

## Hypothesis
<!-- 预期看到什么结果？如果没看到说明什么？人写。-->

## Design
<!-- 实验方案。AI 起草，人审阅。-->

## Config
<!-- 关键配置（只记影响结论的变量，不列全部参数）-->

## Results
<!-- AI 从 CSV/TSV 提取填入。人验证。-->

## Analysis
<!-- AI 起草，人审阅。重点：结果说明了什么，而不是数字本身。-->

## Conclusion
<!-- 人写。AI 绝不碰。支持/推翻/部分支持/意外发现，以及为什么。-->

## Next
<!-- 人写。具体的 action items，不能写"进一步分析"。-->
- [ ] action item 1
- [ ] action item 2
```

## 1.3 写入脚本

### scripts/new_exp.py

```python
"""创建新实验文件，自动编号、填日期。"""
import re
from datetime import date
from pathlib import Path

EXPERIMENTS_DIR = Path(__file__).resolve().parent.parent / "experiments"
TEMPLATE_PATH = EXPERIMENTS_DIR / "_template.md"


def next_exp_id() -> int:
    existing = [d.name for d in EXPERIMENTS_DIR.iterdir() if d.is_dir() and d.name.startswith("EXP")]
    nums = []
    for name in existing:
        m = re.match(r"EXP(\d+)", name)
        if m:
            nums.append(int(m.group(1)))
    return max(nums, default=-1) + 1


def main():
    exp_id = next_exp_id()
    hypothesis = input("Hypothesis IDs (e.g. H1,H2): ").strip() or "TBD"
    model = input("Model name: ").strip() or "TBD"
    dataset = input("Dataset name: ").strip() or "TBD"
    desc = input("Short description (for folder name): ").strip()
    folder_name = f"EXP{exp_id:03d}_{desc}" if desc else f"EXP{exp_id:03d}"
    exp_dir = EXPERIMENTS_DIR / folder_name
    exp_dir.mkdir(parents=True, exist_ok=True)

    template = TEMPLATE_PATH.read_text()
    today = date.today().isoformat()
    content = template.replace("{{{id}}}", f"EXP{exp_id:03d}")
    content = content.replace("{{{hypothesis_ids}}}", hypothesis)
    content = content.replace("{{{model}}}", model)
    content = content.replace("{{{dataset}}}", dataset)
    content = content.replace("{{{date}}}", today)

    out_path = exp_dir / "readme.md"
    out_path.write_text(content)
    print(f"Created: {out_path}")


if __name__ == "__main__":
    main()
```

### scripts/status.py

```python
"""扫描所有实验，输出进度概览。"""
import re
from pathlib import Path

RESEARCH_DIR = Path(__file__).resolve().parent.parent
EXPERIMENTS_DIR = RESEARCH_DIR / "experiments"


def parse_frontmatter(text: str) -> dict:
    m = re.match(r"^---\s*\n(.*?)\n---", text, re.DOTALL)
    if not m:
        return {}
    meta = {}
    for line in m.group(1).strip().splitlines():
        if ":" in line:
            k, v = line.split(":", 1)
            meta[k.strip()] = v.strip()
    return meta


def parse_next_items(text: str) -> list:
    in_next = False
    items = []
    for line in text.splitlines():
        if line.strip().startswith("## Next"):
            in_next = True
            continue
        if in_next and line.startswith("## "):
            break
        if in_next and re.match(r"^\s*-\s*\[.\]", line):
            items.append(line.strip())
    return items


def main():
    print("=" * 60)
    print("Research Status Report")
    print("=" * 60)

    print("\n## Experiments\n")
    exp_dirs = sorted([d for d in EXPERIMENTS_DIR.iterdir()
                       if d.is_dir() and not d.name.startswith("_") and not d.name.startswith(".")])
    for d in exp_dirs:
        readme = d / "readme.md"
        if not readme.exists():
            print(f"  {d.name}: NO readme.md")
            continue
        text = readme.read_text()
        meta = parse_frontmatter(text)
        status = meta.get("status", "unknown")
        hypotheses = meta.get("hypotheses", "-")
        model = meta.get("model", "-")
        dataset = meta.get("dataset", "-")

        next_items = parse_next_items(text)
        next_open = [i for i in next_items if "[ ]" in i]

        status_icon = {"done": "[x]", "active": "[ ]", "paused": "[~]"}.get(status, "[?]")
        print(f"  {status_icon} {d.name}")
        print(f"    status={status}, model={model}, dataset={dataset}, hypotheses={hypotheses}")
        if next_open:
            print(f"    Next items: {len(next_open)}")
            for item in next_open[:3]:
                print(f"      {item}")

    hyp_path = RESEARCH_DIR / "HYPOTHESES.md"
    if hyp_path.exists():
        print("\n## Hypotheses\n")
        text = hyp_path.read_text()
        for line in text.splitlines():
            if line.startswith("## "):
                print(f"  [{line.strip('# ').strip()}]")
            elif line.strip().startswith("- **"):
                print(f"    {line.strip()}")

    roadmap_path = RESEARCH_DIR / "ROADMAP.md"
    if roadmap_path.exists():
        print("\n## Evidence Chain\n")
        text = roadmap_path.read_text()
        in_chain = False
        for line in text.splitlines():
            if "Evidence Chain" in line:
                in_chain = True
                continue
            if in_chain and line.startswith("# "):
                break
            if in_chain and line.strip().startswith("- ["):
                print(f"  {line.strip()}")

    print("\n" + "=" * 60)


if __name__ == "__main__":
    main()
```

## 1.4 冷启动验证

1. 运行 `python research/scripts/new_exp.py` → 确认创建成功
2. 运行 `python research/scripts/status.py` → 确认输出正确
3. 检查所有文件是否创建成功

---

# Part 2: 热更新

> 持续执行。目标：约束 AI 行为，维护实验闭环。

## 2.1 Session 启动协议

每次新 session，AI 必须按顺序执行：

```
1. 读 research/ROADMAP.md         → 当前 story 和证据链
2. 读 research/HYPOTHESES.md      → 假设池状态
3. 读最近 1-2 个实验的 readme.md   → 最新进展
4. 运行 status.py                 → 整体进度
5. 向用户汇报状态摘要，询问要做什么
```

汇报格式：
```
当前状态：
- 活跃实验：EXPXXX（状态：xxx）
- 待验证假设：Hx, Hy
- 证据链进度：x/y 完成
- 未完成 action items：x 个
要做什么？
```

## 2.2 实验闭环

```
[人] 定义问题和假设
  ↓
[AI] python research/scripts/new_exp.py
  ↓
[AI] 起草 Design → [人] 审阅
  ↓
[人] 运行实验
  ↓
[AI] 提取结果填入 Results → [人] 验证
  ↓
[AI] 起草 Analysis → [人] 审阅
  ↓
[人] 写 Conclusion
  ↓
[人] 写 Next action items
  ↓
[AI] 更新 HYPOTHESES + ROADMAP → [人] 审阅
  ↓
[人] 决定下一步
  ↓
└──→ 回到"定义问题和假设"
```

## 2.3 AI 行为约束

### 必须做
- 每次 session 启动先执行 2.1 启动协议
- 实验文件的 Results 段用实际数据填充，不能留 placeholder
- 更新假设状态时同步更新 ROADMAP 的证据链
- 实验文件的 frontmatter `updated` 字段每次改动都要更新

### 绝对不做
- 写 Conclusion 段（研究判断，必须人做）
- 判断假设是否成立（必须人确认）
- 在没有 Question 和 Design 的情况下开始写代码
- 删除或覆盖已有实验文件（只能追加/修改）

### 遇到歧义时
- 停下来，把歧义列出来，问人
- 不要自己猜，不要"先做着看"
- 格式：「这里有 N 个选择：A...B...C...你选哪个？」

## 2.4 实验状态机

每个实验有明确的状态流转：

```
active → done     （人写 Conclusion + Next 后标记）
active → paused   （人决定暂停）
paused → active   （人决定继续）
done   → 不可逆   （不能改回 active）
```

AI 只能在人明确指示后修改状态。不能自己判断"这个实验做完了"。

## 2.5 多实验并行

当有多个 active 实验时：
- 优先级由 ROADMAP.md 的 Current Priority 决定
- 如果没有明确优先级，问人
- 不能自己决定"先做哪个"

## 2.6 异常处理

| 情况 | AI 行为 |
|------|---------|
| 实验脚本报错 | 分析错误，报告给人，不自行修改实验逻辑 |
| 结果看起来不对 | 标注疑问，让人验证，不自行修改数据 |
| Design 执行中发现不可行 | 停下来，说明原因，问人怎么办 |
| status.py 解析失败 | 手动读文件提取信息，报告解析问题 |
| 人给了模糊指令 | 列出可能的理解方式，问人确认 |

---

# Part 3: 通用规范

## 3.1 Git 规范

| 事件 | Commit Message |
|------|---------------|
| 新建实验 | `exp: create EXP003 - 简短描述` |
| 实验完成 | `exp: EXP003 done - 简短结论` |
| 更新 ROADMAP | `roadmap: 更新证据链/优先级` |
| 更新假设 | `hypothesis: H3 verified/rejected` |
| 周回顾 | `weekly: YYYY-WXX` |

规则：
- running 状态的实验不 commit（除非主动保存中间状态）
- 实验 done 时一次性 commit 整个实验文件
- ROADMAP 大幅更新时单独 commit

## 3.2 文件命名规范

- 实验目录：`EXP{编号}_{简短描述}`，如 `EXP003_lowfreq_bias`
- 实验 readme：每个实验目录下固定叫 `readme.md`
- 脚本：放在实验目录下的 `scripts/` 子目录
- 结果：放在实验目录下的 `results/` 子目录

## 3.3 五个铁律

1. **AI 绝不写 Conclusion** — 结论是研究判断，必须由人完成
2. **每个实验必须有 Question 和 Design** — 不能"跑跑看"
3. **Next 要具体** — 不能写"进一步分析"，必须是可执行的 action item
4. **结果数据人必须验证一次** — AI 提取，人验证
5. **新假设必须关联实验** — 不能悬空，必须指向具体 EXP
