---
name: mark-essay
description: 端到端批改一份 0509/23 中文作文 response——用 Claude-in-Chrome 连上已登录的 RM assessor、读两篇作文、按 rubric 评分、用 anchor 校准、稀疏标注、录 S&A/C&S 分、其余题填 NR、过 saving checklist 后保存。当用户要「批改一份作文 / mark a response / 给某个 response ID 打分 / 继续批 worklist」时使用。
---

# 批改一份 response(mark-essay)

端到端流程,把知识库(`.claude/rules/`)和浏览器操作串起来。评分标准本身不在这里——**评分前先读** `rules/rubric.md`、`rules/anchors.md`、`rules/paper-structure.md`、`rules/annotation.md`。

## 0. 前置:连上 RM assessor

- 先调 `tabs_context_mcp` 看用户当前 Chrome tab。
- **复用用户已登录 `https://ca.assessor.rm.com/` 的 session**——不要自己走登录流程、不要输账号密码。若没有已登录的 tab,停下让用户先登录。
- 需要新 tab 时用 `tabs_create_mcp`,不要劫持用户正在用的 tab。

## 1. 取件

- 打开 worklist(`June 2026: 0509/23 WRITING 23: Whole Paper`)。
- 选一个待批 response,或用用户给的 response ID。

## 2. 读作文

- 读扫描件,**两篇作文都读**。手写中文可正常读出。
- 找出每个 section 学生答的题号(看「请把所选题号写在这里」格)。
  - Section 1 ∈ {Q1,Q2,Q3,Q4};Section 2 ∈ {Q5,Q6,Q7,Q8}。

## 3. 评分

对每篇作文:
- 按 `rules/rubric.md` 给 `S&A /12` 和 `C&S /13`,两个 sub-mark 各自独立判。
- 用 `rules/anchors.md` 的 A–J 校准;拿不准 band 时调出相邻分数的 anchor PDF 对比。
- 遵守硬约束(见根 `CLAUDE.md`):不因长给高分;`11–12/13` 只给一贯成熟、展开充分的;选对题但松散切题则封顶 C&S。

## 4. 标注

- 按 `rules/annotation.md` **稀疏但有代表性**地标证据,不逐行标。
- judgement call 处正负标记都放几个。
- 空白 / 已读未标的页用 `SEEN`。

## 5. 录分

- 被选的 Section 1 题:录 `S&A /12` + `C&S /13`。
- 被选的 Section 2 题:录 `S&A /12` + `C&S /13`。
- 两个 section 其余可选题全部 `NR`。

## 6. 校验(过 saving checklist)

逐条对 `rules/paper-structure.md` 的存档前 checklist:
- 每 section 恰好一题被评分;其余全 `NR`。
- 被选题都有双 sub-mark;题分 = 两 mark 之和 `/25`。
- 总分 = 两 section 之和 `/50`。

## 7. 保存

- **状态对了才存**:停在错题上、或缺某个 sub-mark 时,绝不保存。
- 用户现有 workflow 多为先存 **draft**;除非用户要求提交,默认存 draft 并报告分数明细让用户确认。

## 浏览器纪律

- **不要触发** alert / confirm / prompt 弹窗——会卡死扩展,导致后续命令收不到。遇到带确认弹窗的按钮先警告用户。
- 同一浏览器操作失败 2–3 次、或扩展无响应,就**停下来问用户**,不要反复重试或乱点。
- 多步关键操作(录分、保存)可用 `gif_creator` 录制留证,文件名起得能认出来(如 `mark-6157733599.gif`)。
- 报告时给出:response ID、每 section 的题号与 `S&A+C&S`、总分、以及 NR 项,让用户复核。
