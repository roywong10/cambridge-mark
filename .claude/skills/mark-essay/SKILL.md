---
name: mark-essay
description: 端到端批改一份 0509/23 中文作文 response——用 Claude-in-Chrome 连上已登录的 RM assessor、通读两篇作文、按 rubric 评分、用 anchor 校准、列标注清单、一次性批量录 S&A/C&S 分 + 标注、其余题填 NR、过 saving checklist 后 Save(永不 Submit)。当用户要「批改一份作文 / mark a response / 给某个 response ID 打分 / 继续批 worklist」时使用。
---

# 批改一份 response(mark-essay)

端到端流程,把知识库(`.claude/rules/`)和浏览器操作串起来。评分标准本身不在这里——**评分前先读** `rules/rubric.md`、`rules/anchors.md`、`rules/paper-structure.md`、`rules/annotation.md`。

## 核心流程原则:读+决策 与 执行 分离

**不要边滚边点**——那样每步都要滚动+截图+点击,在 16 页长文档上极慢且易点错。正确做法分两段:

1. **读 + 决策(不碰标注工具)**:用缩略图视图一次性通读两篇 → 在回复里写下完整的「评分 + 标注清单」(每条:页码 + 位置 + 标记类型)。
2. **批量执行**:按清单跳页,把分数和标注一次性填上,减少来回滚动。

---

## 0. 前置:连上 RM assessor

- 先调 `tabs_context_mcp` 看用户当前 Chrome tab。
- **复用用户已登录 `https://ca.assessor.rm.com/` 的 session**——不要自己走登录流程、不要输账号密码。若没有已登录的 tab,停下让用户先登录。
- 需要新 tab 时用 `tabs_create_mcp`,不要劫持用户正在用的 tab。

## 1. 取件

- 打开 worklist(`June 2026: 0509/23 WRITING 23: Whole Paper`)。worklist 分三态:`Open for marking`(待批)/ `Submitted - editable`(已批、grace period 内可改)/ `Submitted - closed`(已锁)。
- 批没批过的:进 `Open for marking`,选 `Marking not started` 的件(或用用户给的 ID)。
- 列表每行显示 `Response ID` + `Progress` + 总分(`Marks`);点 ID 进入批改页。

## 2. 通读两篇(缩略图视图)

- 进批改页后,点**左上角第一个图标**切到**缩略图视图**,选 **4-page** 模式——一屏看 4 页,快速通读、定位。
- 版式提醒:同一份里**繁体题目页 + 繁体答题区**和**简体题目页 + 简体答题区**各一套;学生通常只用一套(另一套全空白)。第一篇(Section 1)在前,第二篇(Section 2)在后,中间夹着第二部分题目页。
- **两篇都读完**。手写中文可正常读出。
- 找出每个 section 学生答的题号(看「请把所选题号写在这里」格,学生会填题号)。
  - Section 1 ∈ {Q1,Q2,Q3,Q4};Section 2 ∈ {Q5,Q6,Q7,Q8}。
- 记下每篇是哪种题型(议论/讨论 vs 描述/叙述)。

## 3. 评分 + 列标注清单(决策阶段,不碰浏览器工具)

对每篇作文:
- 按 `rules/rubric.md` 给 `S&A /12` 和 `C&S /13`,两个 sub-mark 各自独立判。
- 用 `rules/anchors.md` 的 A–J 校准;拿不准 band 时调出相邻分数的 anchor PDF 对比。
- 遵守硬约束(见根 `CLAUDE.md`):不因长给高分;`11–12/13` 只给一贯成熟、展开充分的;选对题但松散切题则封顶 C&S。

然后**在回复里写出完整决策清单**,例如:

```text
评分:
  Section 1 = Q3,S&A 9 + C&S 9 = 18/25(理由…)
  Section 2 = Q8,S&A 8 + C&S 8 = 16/25(理由…)
  Total = 34/50
标注清单:
  每个有内容的页 → SEEN(落点:页眉旁或页底空白格)
  page 11 错别字「来沛」 → H Wavy
  page 11 好论点「…」 → Tick
  page 15 情感结句「…」 → Tick
```

把这份清单**报给用户确认分数**后再进入执行。

## 4. 批量录分(mark 树 UI,实测)

右侧面板是 mark 树,结构:

```text
Section 1 (/25)
  题号 (/25)
    <题号>S&A (/12)
    <题号>C&S (/13)
  …其余题为 NR
Section 2 (/25)
  …同上
Total marks XX/50   [Save]
```

- 顶部:`Mark by: Candidate`、`Annotations: ON`(标注开关)。
- 录分方式:**点选某个 `<题号>S&A` / `<题号>C&S` 字段** → **点最右一列的数字圆点**(`0`–`12` 给 S&A,`0`–`13` 给 C&S,以及 `NR`)选分。
- **录完一个 sub-mark,UI 会自动跳到下一个字段**——顺势点下一个数字即可,不必每次手动选字段。
- 顺序:点被选题的 `S&A` 字段 → 点数字 → (自动跳 `C&S`)点数字。Section 1、Section 2 各一遍。
- 两个 section 其余可选题全部留 `NR`(mark 树里显示为划线项,**不去点它们**)。
- 录分时**只点目标字段的圆点**;只读 / 复核别人的件时,绝不误点任何圆点或 `Reset`(会改分)。

## 5. 批量标注

按第 3 步的标注清单逐页执行。**加标注**:点左侧工具激活 → 点页面落点。

左侧工具(自上而下,红色组):`SEEN` / `?` / `H Wavy`(波浪线)/ `Highlight` / `^` / `Tick` / `DET`。细节见 `rules/annotation.md`。

- **SEEN 基本每页都打**,落点选**空白处**(页眉旁、页底空白格),**别压在字上**。
- 证据标记(`H Wavy` / `Tick` / `DET` 等)稀疏、有代表性,落在对应的字/句上。
- **删单个标注**:**右键标注 → 点 `Remove annotation`**。
- ⚠️ **危险**:同一右键菜单里还有 **`Reset marks & annotations`**——它会**清空整题的分数 + 所有标注**。**绝不能误点它的 `Yes`**;误弹出时点 `No`。删单个标注只用 `Remove annotation`,不要用它。

## 6. 校验(过 saving checklist)

逐条对 `rules/paper-structure.md` 的存档前 checklist:
- 每 section 恰好一题被评分;其余全 `NR`。
- 被选题都有双 sub-mark;题分 = 两 mark 之和 `/25`。
- 总分 = 两 section 之和 `/50`。

## 7. 保存(只 Save,永不 Submit)

- **状态对了才存**:停在错题上、或缺某个 sub-mark 时,绝不保存。
- 点底部 **`Save`** 存分,response 停在 `Open for marking`(Progress 25% 即两题已录),grace period 内可改。
- **`Save` ≠ `Complete` ≠ `Submit`**:底部除 `Save` 外还有 `Complete`/`Submit`。**只点 `Save`**,绝不点 `Complete` / `Submit`(见根 `CLAUDE.md` 硬约束 8)——那些把分数返还 awarding body、需要人来 review,只能由人来点。
- Save 后回 `Worklist` 面包屑,确认该 ID 的 `Marks` 列显示了刚录的总分、且仍在 `Open for marking`,再报告给用户复核。

## 浏览器纪律

- **不要触发** alert / confirm / prompt 弹窗——会卡死扩展,导致后续命令收不到。遇到带确认弹窗的按钮先警告用户。
- **批改页有通用「Leave site?」离开保护**:每个 response 页用 URL 导航离开时都会被拦,**这不代表有未保存更改**。回 worklist 走页面顶部的 `Worklist` 面包屑链接(应用内导航),不要用 `force` 直接导航——以免在真有未保存分数时丢弃。
- 同一浏览器操作失败 2–3 次、或扩展无响应,就**停下来问用户**,不要反复重试或乱点。
- 多步关键操作(录分、保存)可用 `gif_creator` 录制留证,文件名起得能认出来(如 `mark-6157733599.gif`)。
- 报告时给出:response ID、每 section 的题号与 `S&A+C&S`、总分、以及 NR 项,让用户复核。
