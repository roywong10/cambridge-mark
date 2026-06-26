---
name: mark-essay
description: 端到端批改一份 0509/23 中文作文 response——用 Claude-in-Chrome 连上已登录的 RM assessor、通读两篇作文、按 rubric 评分、用 anchor 校准、录 S&A/C&S 分 + 其余题 NR、Save(永不 Submit / Complete),并把打分理由 + 措辞/语法问题 + 错别字写成一节追加到汇总报告。当用户要「批改一份作文 / mark a response / 给某个 response ID 打分 / 继续批 worklist / 自动批」时使用。
---

# 批改一份 response(mark-essay)

把知识库(`.claude/rules/`)和浏览器操作串起来。评分标准本身不在这里——**评分前先读** `rules/rubric.md`、`rules/anchors.md`、`rules/paper-structure.md`。

## 工作分工(重要)

- **浏览器里只做一件事:录分**(S&A/C&S + 其余 NR + Save)。**不做任何 annotation**(不打 SEEN、不画波浪线、不标错别字)——RM viewer 的逐页标注交互成本高、落点难,实测除 SEEN 外难以稳定点准,SEEN 也省了。
- **分析落在报告里**:打分理由、措辞/语法问题、错别字,写成一节追加到汇总报告 `marking-reports/0509-23-writing-23.md`(该目录已 gitignore,不进 git)。
- 浏览器可靠地做它擅长的(点数字录分),文档承载分析。

## 0. 前置:连上 RM assessor

- 先调 `tabs_context_mcp` 看用户当前 Chrome tab。
- **复用用户已登录 `https://ca.assessor.rm.com/` 的 session**——不要自己登录、不输账号密码。没有已登录 tab 就停下让用户先登录。

## 1. 取件

- worklist(`June 2026: 0509/23 WRITING 23: Whole Paper`)三态:`Open for marking`(待批)/ `Submitted - editable`(已批可改)/ `Submitted - closed`(已锁)。
- 批没批过的:进 `Open for marking`,选 `Marking not started` 的件。自动模式下逐个往下批。
- 列表每行:`Response ID` + `Progress` + `Marks`;点 ID 进批改页。

## 2. 通读两篇(缩略图视图)

- 进批改页后点**左上角第一个图标**切**缩略图视图**(2-page / 4-page),一屏多页快速通读、定位。
- 版式:繁体题目+答题区 与 简体题目+答题区 各一套,学生通常只用一套(另一套空白)。第一篇(Section 1)在前,第二篇(Section 2)在后,中间夹第二部分题目页。
- **两篇都读完**。手写中文可正常读出;拿不准的字用 `zoom` 放大。
- 找出每 section 学生答的题号(看「请把所选题号写在这里」格)。Section 1 ∈ {Q1–Q4};Section 2 ∈ {Q5–Q8}。
- 通读时**顺手记下**:措辞/语法问题(在第几篇、什么问题)、错别字(具体哪个字、在第几篇)——这些进报告。

## 3. 评分(决策)

对每篇:
- 按 `rules/rubric.md` 给 `S&A /12` + `C&S /13`,两 sub-mark 各自独立判。
- 用 `rules/anchors.md` 的 A–J 校准;拿不准 band 时调相邻分数 anchor PDF 对比。
- 遵守硬约束(根 `CLAUDE.md`):不因长给高分;`11–12/13` 只给一贯成熟、展开充分的;选对题但松散切题封顶 C&S;两篇逐篇独立判。
- 自动模式下**自主判分,不每篇问用户**。

## 4. 录分(mark 树 UI)

右侧 mark 树:`Section N (/25) → 题号 (/25) → <题号>S&A (/12) + <题号>C&S (/13)`,底部 `Total marks XX/50` + `Save`。

- **点选 `<题号>S&A` 字段 → 点最右一列数字圆点选分**。录完 S&A **自动跳 C&S**,顺势点数字。
- Section 1、Section 2 各一遍(被选题)。其余可选题留 `NR`(显示为划线,**不去点**)。
- 只点目标字段圆点;**绝不右键**(右键有 `Reset marks & annotations` 会清空整题)。

## 5. 校验

- 每 section 恰好一题有分,其余 NR;题分 = 两 sub-mark 之和 `/25`;总分 = 两 section 之和 `/50`。

## 6. 保存(只 Save,永不 Submit / Complete)

- 状态对了才存。点底部 **`Save`**,response 停在 `Open for marking`(Progress 25% = 两题已录)。
- **绝不点 `Complete` / `Submit`**(根 `CLAUDE.md` 硬约束)——那些返还 awarding body、需人 review,只能人点。
- Save 后回 `Worklist` 面包屑确认该 ID 的 `Marks` 列显示了总分。

## 7. 写报告(追加一节到汇总)

把这篇追加为一节到 `marking-reports/0509-23-writing-23.md`,含:

```markdown
## <Response ID> — <总分>/50

| Section | 题 | S&A | C&S | 小计 |
|---|---|---:|---:|---:|
| 1 | Q<x> <题型> | a | b | a+b |
| 2 | Q<y> <题型> | c | d | c+d |

**Section 1 — Q<x>(a + b)**
- 打分理由:…(为何这个 band)
- 措辞/语法问题:具体几个,各自在第几篇/哪句
- 错别字:几个,是什么字,在第几篇/哪处

**Section 2 — Q<y>(c + d)**
- 打分理由:…
- 措辞/语法问题:…
- 错别字:…
```

## 8. 自动模式(全自动不间断)

用户要「自动批 / 一篇一篇批」时:循环 —— 取下一个 `Marking not started` 件 → 通读 → 评分 → 录分 → Save → 写报告一节追加到汇总 → 下一篇。**自主判分不每篇问**;批完所有待批件或用户叫停后,报告汇总情况。中途遇到需人判断的(疑似抄袭/违规线索)才停下来问。

## 浏览器纪律

- **不触发** alert/confirm/prompt 弹窗(卡死扩展)。**绝不右键标注**(Reset 危险)。
- 批改页有通用「Leave site?」离开保护——回 worklist 走顶部 `Worklist` 面包屑(应用内导航),不用 `force` 直接导航。
- 单页视图往上滚动受限;通读/定位优先用**缩略图视图**(可滚全貌)。
- 同一操作失败 2–3 次或扩展无响应就**停下问用户**,不反复重试乱点。
- 每篇报告里给:Response ID、每 section 题号与 `S&A+C&S`、总分,以及问题清单。
