# cambridge-mark

一个 **Claude Code 知识库 harness**,用来批改 **Cambridge IGCSE `0509/23` First Language Chinese, Paper 2 Writing**(中文手写作文)。

这个 repo **没有运行代码**——它全部是文档和 marking rule。批改本身由 Claude Code 完成:它读这里的知识库,然后用 **Claude-in-Chrome 连接器**登录 [`https://ca.assessor.rm.com/`](https://ca.assessor.rm.com/)(复用你已登录的浏览器 session),按知识库标准给作文评分。

## 怎么用

1. 在浏览器里登录 RM assessor(`ca.assessor.rm.com`)。
2. 在本目录跑 Claude Code。
3. 让它批改:给一个 response ID,或让它从 worklist 取件。

Claude 会:读两篇作文 → 识别每个 section 的选题 → 按 rubric 给 `S&A /12` + `C&S /13` → 用 anchor 校准 → 稀疏标注 → 其余题填 `NR` → 过 saving checklist → 存 draft 并报告分数让你复核。

## 结构

- **`CLAUDE.md`** — agent 入口:任务定位、硬约束、路由表。**先读这个。**
- **`.claude/rules/`** — 评分知识真源:
  - `paper-structure.md` — 试卷结构、选题、NR、录分流程、saving checklist
  - `rubric.md` — S&A /12 + C&S /13 各 band 描述
  - `anchors.md` — A–J 校准锚点 + calibration rules
  - `annotation.md` — RM 标注工具用法与纪律
- **`.claude/skills/mark-essay/`** — 端到端批改流程(含 Chrome 操作)。

## 注意

官方示例 PDF(A–J,redacted 学生作文)是**本地外部资源**,在 `~/Downloads/0509_2320260527221836/`,不进 repo、不进 git。
