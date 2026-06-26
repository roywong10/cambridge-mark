# CLAUDE.md

AI 助手在 `cambridge-mark` 的入口。先读本文件,再按任务打开相关 rule / skill。

## 这是什么

这是一个**知识库 harness**——没有运行代码,全部是文档 + marking rule。

用途:Claude 用 **Claude-in-Chrome 连接器**登录 `https://ca.assessor.rm.com/`(复用已登录的 session),按本知识库标准批改 **Cambridge `0509/23` First Language Chinese, Paper 2 Writing**——手写中文作文,扫描件。

试卷一句话:总分 `50`;Section 1 `/25` + Section 2 `/25`;每个被选题 = `S&A /12` + `C&S /13`;每 section 选一题,其余 `NR`。

## 硬约束 — ALWAYS apply

这些是最常被违反、且直接决定批改对错的纪律,放在入口里。

1. **每 section 只给一题评分**,其余可选题一律 `NR`。选错题 / 多给题是最常见错误。
2. **每个被选题必须同时有 `S&A /12` 和 `C&S /13`**,缺一个 sub-mark 绝不保存。
3. **不因篇幅长就给高分**。长但重复 / 松散 → C&S 落 `7–8`,不是 `9–10`。
4. **录分前用 anchor(A–J)校准**。`11–12/13` 只给一贯成熟、展开充分的脚本。
5. **标注稀疏但有代表性**,服务评分证据,不逐行标。
6. **存档前过 saving checklist**(单题 / NR / 双 mark / 题分与总分一致)。
7. **绝不替学生改作文;绝不在错误状态下保存**(停在错题、或缺 sub-mark 时不存)。
8. **永远只 `Save`,绝不 `Submit`**。批完改好就 Save,停在 editable 状态。`Submit` 把分数返还给 awarding body、需要人来 review,**只能由人来点**——agent 任何情况下都不点 Submit。
9. **浏览器纪律**:复用用户已登录 session(不自己登录);不触发 alert/confirm 弹窗(会卡死扩展);操作失败 2–3 次就停下问用户。

## 任务路由

| 你要做… | 读 / 用 |
|---|---|
| 批改一份 response(连 Chrome → 读 → 评分 → 标注 → 录分 → 存)| skill `mark-essay` |
| 看试卷结构、选题规则、NR、录分流程、saving checklist | `.claude/rules/paper-structure.md` |
| 给作文定 band / 定分(S&A /12、C&S /13 各 band 描述)| `.claude/rules/rubric.md` |
| 用 anchor 校准、或两个 band 之间拿不准 | `.claude/rules/anchors.md` |
| 在 RM 工具里加标注 | `.claude/rules/annotation.md` |

## Source-of-truth 原则

- 评分标准的真源在 `.claude/rules/`,各归一处不重复:结构/流程 → `paper-structure.md`,band → `rubric.md`,校准 → `anchors.md`,标注 → `annotation.md`。
- 端到端操作流程在 skill `mark-essay`。
- **anchor 官方示例 PDF 是本地外部资源**,在 `~/Downloads/0509_2320260527221836/`,不在 repo 内、不进 git。详见 `.claude/rules/anchors.md`。
