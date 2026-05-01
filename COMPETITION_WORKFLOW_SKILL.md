# Competition Workflow Skill — 竞赛迭代工作流

> AI 协同的竞赛迭代工作流。两种模式：冷启动搭建 + 热更新运行。

---

## 模式判断

AI 读到这个文件后，第一步：检查 `competition/` 目录是否存在且包含 `ROADMAP.md`。

- **不存在或不完整** → 执行「冷启动」
- **存在且完整** → 执行「热更新」

---

# Part 1: 冷启动

> 一次性执行。目标：从零搭建完整的迭代管理基础设施。

## 1.1 创建目录结构

```
competition/
├── BOOTSTRAP.md          # AI 启动手册
├── ROADMAP.md            # 迭代路线 + 方案追踪
├── LEADERBOARD.md        # 分数追踪
├── RESPONSIBILITY.md     # 职责分工
├── scripts/
│   ├── new_exp.py        # 创建实验
│   └── status.py         # 扫描进度
└── experiments/
    ├── _template.md      # 实验模板
    └── EXP001_baseline/  # 首个实验：baseline 复现
        └── readme.md
```

## 1.2 写入文件

### BOOTSTRAP.md

```markdown
# 竞赛迭代 — AI Onboarding

## 项目背景
<!-- 人填：竞赛名称、任务描述、评估指标、当前 baseline 分数 -->

## 启动流程（每次新 session 执行）
1. 读 competition/ROADMAP.md → 了解当前版本和迭代计划
2. 读 competition/LEADERBOARD.md → 了解当前最优分数
3. 读最近 1-2 个实验的 readme.md → 了解最新进展
4. git log --oneline -5 → 最近提交
5. 运行 python competition/scripts/status.py → 整体进度
6. 告诉用户当前状态摘要，询问要做什么

## 关键路径
- baseline 代码：baseline/（只读，禁止修改）
- 开发代码：src/
- 实验记录：competition/experiments/

## 关键规则
- baseline/ 绝不修改，改动全部在 src/ 进行
- 每个实验必须有 Question 和 Design，不能"跑跑看"
- Next 里的 action items 要具体，不能写"进一步分析"
- 结果数据由 AI 提取，但人必须验证一次
- 版本发布（合 main + 打 tag）必须人确认
```

### ROADMAP.md

```markdown
# 竞赛目标
<!-- 人填：目标分数或排名 -->

# 当前版本
<!-- 当前最高分、对应实验编号 -->

| 版本 | 实验 | 方案摘要 | 最优指标 | tag |
|------|------|---------|---------|-----|
| v0.1 | EXP001 | baseline 复现 | - | v0.1 |

# 迭代计划
<!-- 每个条目关联具体实验编号 -->
- [ ] v0.2: 方案描述 → EXP002
- [ ] v0.3: 方案描述 → EXP003

# 已走过的路
<!-- 记录失败方案和原因，避免重复踩坑 -->
```

### LEADERBOARD.md

```markdown
# 分数追踪

| 版本 | 实验 | 方案摘要 | 指标A | 指标B | 备注 |
|------|------|---------|-------|-------|------|
| v0.1 | EXP001 | baseline | - | - | 官方 baseline 复现 |
```

### RESPONSIBILITY.md

```markdown
# 职责分工

## AI 独立做
- [✓] 创建/编号实验文件（new_exp.py）
- [✓] 从日志/CSV 提取结果数据填入实验 Results
- [✓] 从配置文件提取实验参数填入 Config
- [✓] 生成 status 报告
- [✓] 更新 LEADERBOARD.md
- [✓] 起草 Design / Analysis 段落

## AI 起草，人必须审阅
- [◐] 实验设计方案（Design）
- [◐] Analysis（分数变化分析）
- [◐] ROADMAP 更新
- [◐] 版本合入 main 的决定

## 人独立做
- [✗] 决定实验方向和方案
- [✗] 写代码改模型/特征/训练策略
- [✗] 决定版本发布（合 main + 打 tag）
- [✗] 判断是否继续迭代或切换方向
```

### experiments/_template.md

```markdown
---
id: {{{id}}}
status: active
version: {{{version}}}
parent: {{{parent}}}
scheme: {{{scheme}}}
created: {{{date}}}
updated: {{{date}}}
---

## Question
<!-- 这个实验要验证什么方案？一句话。-->

## Hypothesis
<!-- 预期看到什么结果？如果没看到说明什么？-->

## Design
<!-- 实验方案：改了什么，为什么改，预期效果。-->

## Config
<!-- 关键配置（只记影响结论的变量，不列全部参数）-->

## Results
<!-- AI 从日志/CSV 提取填入。人验证。-->

## Analysis
<!-- AI 起草：分数变化了多少，可能的原因是什么。-->

## Conclusion
<!-- 人写：这个方案是否有效，下一步怎么走。-->

## Next
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
    scheme = input("Scheme description (一句话): ").strip() or "TBD"
    parent = input("Parent version (e.g. v0.1 or none): ").strip() or "none"
    version = input("This version (e.g. v0.2): ").strip() or f"v0.{exp_id}"
    desc = input("Short description (for folder name): ").strip()
    folder_name = f"EXP{exp_id:03d}_{desc}" if desc else f"EXP{exp_id:03d}"
    exp_dir = EXPERIMENTS_DIR / folder_name
    exp_dir.mkdir(parents=True, exist_ok=True)

    template = TEMPLATE_PATH.read_text()
    today = date.today().isoformat()
    content = template.replace("{{{id}}}", f"EXP{exp_id:03d}")
    content = content.replace("{{{version}}}", version)
    content = content.replace("{{{parent}}}", parent)
    content = content.replace("{{{scheme}}}", scheme)
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

COMP_DIR = Path(__file__).resolve().parent.parent
EXPERIMENTS_DIR = COMP_DIR / "experiments"


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
    print("Competition Iteration Status")
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
        version = meta.get("version", "-")
        scheme = meta.get("scheme", "-")

        next_items = parse_next_items(text)
        next_open = [i for i in next_items if "[ ]" in i]

        status_icon = {"done": "[x]", "active": "[ ]", "paused": "[~]"}.get(status, "[?]")
        print(f"  {status_icon} {d.name}")
        print(f"    status={status}, version={version}, scheme={scheme}")
        if next_open:
            print(f"    Next items: {len(next_open)}")
            for item in next_open[:3]:
                print(f"      {item}")

    leaderboard_path = COMP_DIR / "LEADERBOARD.md"
    if leaderboard_path.exists():
        print("\n## Leaderboard\n")
        text = leaderboard_path.read_text()
        for line in text.splitlines():
            if line.strip().startswith("|") and not line.strip().startswith("| --"):
                print(f"  {line.strip()}")

    print("\n" + "=" * 60)


if __name__ == "__main__":
    main()
```

## 1.4 冷启动验证

1. 运行 `python competition/scripts/new_exp.py` → 确认创建成功
2. 运行 `python competition/scripts/status.py` → 确认输出正确
3. 检查所有文件是否创建成功

---

# Part 2: 热更新

> 持续执行。目标：维护迭代闭环，追踪实验进度和分数。

## 2.1 Session 启动协议

每次新 session，AI 必须按顺序执行：

```
1. 读 competition/ROADMAP.md         → 当前版本、目标、迭代计划
2. 读 competition/LEADERBOARD.md     → 当前最优分数
3. 读最近 1-2 个实验的 readme.md   → 最新进展
4. git log --oneline -5           → 最近提交
5. 运行 python competition/scripts/status.py → 整体进度
6. 向用户汇报状态摘要，询问要做什么
```

汇报格式：
```
当前状态：
- 当前版本：v0.x（最高分：xxx）
- 活跃实验：EXPXXX（方案：xxx）
- 未完成 action items：x 个
- 最近提交：xxx
要做什么？
```

## 2.2 实验闭环

```
[人] 决定方案
  ↓
[AI] python competition/scripts/new_exp.py
  ↓
[AI] 起草 Design → [人] 审阅
  ↓
[人] 写代码 / 跑实验
  ↓
[AI] 提取结果填入 Results → [人] 验证
  ↓
[AI] 起草 Analysis → [人] 审阅
  ↓
[人] 写 Conclusion + Next
  ↓
[AI] 更新 LEADERBOARD + ROADMAP → [人] 审阅
  ↓
[人] 决定：继续迭代 / 合 main 打 tag / 切换方向
  ↓
└──→ 回到"决定方案"
```

## 2.3 Git 版本控制策略

### 分支模型

```
main    ← 稳定版本，只有验证通过的才合入，每个版本打 tag
  └── dev  ← 日常开发，所有实验在这里进行
```

- **main**：只放经过验证的版本，合入时打 tag（v0.1, v0.2...）
- **dev**：日常实验的集成分支，所有实验 commit 在这里

### 版本发布流程

```
[人] 决定当前实验结果可以发布
  ↓
[AI] git add + commit 实验文件（dev 分支）
  ↓
[AI] git checkout main && git merge dev
  ↓
[AI] git tag v0.x
  ↓
[AI] git checkout dev（回到开发分支）
```

注意：合 main + 打 tag 必须人确认，AI 不能自行发布。

### 查看版本间变化

```bash
# 查看两个版本之间的所有变化
git diff v0.1..v0.2

# 查看某个版本的实验记录
git show v0.2:competition/experiments/EXP002_xxx/readme.md
```

## 2.4 AI 行为约束

### 必须做
- 每次 session 启动先执行 2.1 启动协议
- 实验文件的 Results 段用实际数据填充，不能留 placeholder
- 更新 LEADERBOARD 时保持格式一致
- 实验文件的 frontmatter `updated` 字段每次改动都要更新
- commit 前检查是否有未保存的实验数据

### 绝对不做
- 修改 baseline/ 目录下的任何文件
- 自行决定合 main 或打 tag（必须人确认）
- 在没有 Question 和 Design 的情况下开始跑实验
- 删除或覆盖已有实验文件（只能追加/修改）
- 自行决定放弃某个方案（必须人确认）

### 遇到歧义时
- 停下来，把歧义列出来，问人
- 不要自己猜，不要"先做着看"
- 格式：「这里有 N 个选择：A...B...C...你选哪个？」

## 2.5 实验状态机

每个实验有明确的状态流转：

```
active → done     （人写 Conclusion + Next 后标记）
active → paused   （人决定暂停）
paused → active   （人决定继续）
done   → 不可逆   （不能改回 active）
```

AI 只能在人明确指示后修改状态。不能自己判断"这个实验做完了"。

## 2.6 异常处理

| 情况 | AI 行为 |
|------|---------|
| 实验脚本报错 | 分析错误，报告给人，不自行修改实验逻辑 |
| 结果看起来不对 | 标注疑问，让人验证，不自行修改数据 |
| Design 执行中发现不可行 | 停下来，说明原因，问人怎么办 |
| status.py 解析失败 | 手动读文件提取信息，报告解析问题 |
| 人给了模糊指令 | 列出可能的理解方式，问人确认 |
| 分数下降 | 报告变化，不自行判断是否继续，问人决定 |

---

# Part 3: 通用规范

## 3.1 Git Commit 规范

| 事件 | Commit Message | 示例 |
|------|---------------|------|
| 新建实验 | `exp: create EXP003 - 简短描述` | `exp: create EXP003 - lr_schedule` |
| 实验完成 | `exp: EXP003 done - 指标变化` | `exp: EXP003 done - +0.5% NDCG` |
| 版本发布 | `release: v0.2 - 简短描述` | `release: v0.2 - add lr schedule` |
| 更新路线图 | `roadmap: 更新迭代计划` | |
| 更新分数 | `leaderboard: EXP003 结果` | |

规则：
- running 状态的实验不 commit（除非主动保存中间状态）
- 实验 done 时 commit 整个实验文件
- 版本发布时在 main 分支打 tag

## 3.2 文件命名规范

- 实验目录：`EXP{编号}_{简短描述}`，如 `EXP003_lr_schedule`
- 实验 readme：每个实验目录下固定叫 `readme.md`
- 脚本：放在实验目录下的 `scripts/` 子目录
- 结果：放在实验目录下的 `results/` 子目录

## 3.3 四个铁律

1. **baseline/ 绝不修改** — 改动全部在 src/ 进行
2. **每个实验必须有 Question 和 Design** — 不能"跑跑看"
3. **Next 要具体** — 不能写"进一步分析"，必须是可执行的 action item
4. **版本发布必须人确认** — 合 main + 打 tag 不能自动化
