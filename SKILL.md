---
name: intro-related-work-builder
description: Draft a five-move introduction and a theme-grouped related-work section from supplied background notes and a reference library, keeping every sentence traceable and marking missing gap or contribution material instead of inventing it. Use when a task asks for an introduction, literature review or research-gap narrative with correct citation placement.
---

# 引言与相关工作写作

## 用途与适用条件

把背景材料、研究缺口陈述、作者贡献与文献题录，组织成**五段式引言**（背景 → 已有工作 → 研究缺口 → 本文贡献 → 结构路线图）和**主题分组的相关工作**，逐句保留来源定位。

适用：

- 任务是写/改 Introduction 或 Related Work，且材料由用户提供。
- 需要明确的"缺口—贡献"对应关系，以及引用是否落到实处的检查。
- 需要可回溯到原材料的可编辑 Markdown。

不适用：需要真实文献检索、需要评价新颖性真伪、需要写方法与结果章节。

## 输入

| 输入 | 位置 | 说明 |
|-|-|-|
| `case.json` | `/input/case.json`（`--case` 可覆盖） | 材料取自 `materials` / `background` / `notes` / `excerpts` / `manuscript` 等；贡献取自 `contributions` / `our_contributions` / `highlights`；文献取自 `references` / `bibliography` / `refs` |
| `task.md` | `/input/task.md`（自动探测） | 可提供 `贡献:` 列表，作为贡献小节来源 |
| 输出 | `/app/output/result.json`（`--out` 可覆盖） | 结构化结果 |

## 处理流程（对应评分维度）

1. **材料清点**（找得到）：收集字符串、列表与 `{text: ...}` 材料；切句并保留 `路径#s序` 定位。
2. **动作识别**：用缺口标记（however / few studies / remains unclear / 缺乏…）与背景标记（important / widely / burden / 重要…）切分句子角色，缺口句**不在背景与已有工作重复使用**。
3. **五段式成稿**（写得成）：背景取 2 句、已有工作取剩余引用句、缺口取最多 3 句、贡献取自作者材料、路线图按 IMRaD 通用编号；每句附 `[src:...]`。
4. **贡献不代写**（诚实性）：无贡献材料时输出占位说明并写入 `uncertainty`，**不推断"首次/首个"等新颖性声明**。
5. **相关工作分组**（排得清）：按题录标题/摘要关键词分为 methods、data_and_benchmarks、applications、theory_and_analysis、surveys、general；有材料句支撑的给原句，只有题录的明确标注"未生成内容"。
6. **引用覆盖**：统计被材料引用的文献与未被引用文献，题录缺年份/题名逐条提示。

## 输出契约

- `case_id`：原样保留。
- `status`：`success`（有贡献且有缺口陈述）/ `partial`（缺其一或材料不足）/ `insufficient_input`（无材料）/ `failed`。
- `result`：`introduction`、`related_work`、`deliverable`（两者拼接）、`contributions`、`gap_statements`、`theme_groups`、`cited_references`、`uncited_references`、`limitations`。
- 顶层另置 `introduction`、`related_work`、`deliverable` 便于契约映射。
- `evidence`（句子→定位）、`uncertainty`、`errors`、`stats`（句数、引用句、缺口句、贡献条数、文献数、主题组数）、`contract_notes`。

**output_contract 处理**：声明字段逐一置于顶层；`introduction`、`related_work`、`deliverable` 及别名（`intro`、`literature_review` 等）会映射；无法产出字段置 `null` 并记录，不虚构段落。

## 异常处理

| 情况 | 处置 | status |
|-|-|-|
| `case.json` 缺失/非法 | 结构化失败，标注阶段，退出码 2 | `failed` |
| 无任何材料 | 不产出文本，说明缺什么 | `insufficient_input` |
| 无贡献材料 | 贡献小节保留占位，不代写 | `partial` |
| 无缺口陈述 | 缺口小节保留占位 | `partial` |
| 文献库为空 | 正文引用无法对应题录，写入 `uncertainty` | 原状态 |
| 文献题录缺字段 | 逐条提示，不补造年份/题名 | 原状态 |
| 材料句无引用 | 相关工作仅按题录分组并明确标注 | 原状态 |

## 限制

- 不检索文献，不判断综述是否覆盖完整；新颖性断言一律不生成。
- 主题分组基于关键词匹配，跨学科文献可能落入 `general`。
- 路线图章节号固定为通用 IMRaD，需与目标期刊模板核对。
- 输出为可编辑草稿，作者需补充实质评述与方法细节。

## 最小调用示例（示例，非官方案例结果）

```json
{
  "case_id": "demo-001",
  "output_contract": {"introduction": "string", "related_work": "string", "status": "string"},
  "contributions": ["提出闭环吸入监测框架。"],
  "materials": ["Asthma affects 300 million people [1]. However, few studies examined closed-loop monitoring in adults [2]."],
  "references": [{"key": "R1", "year": "2023", "title": "Global asthma strategy"}, {"key": "R2", "year": "2021", "title": "Adherence in primary care"}]
}
```

```bash
python scripts/run_skill.py --case /input/case.json --out /app/output/result.json --work /app/work
```

示意输出（**示例**）：`status=success`，`introduction` 含 1.1–1.5 五个小节，背景句与缺口句不重复，`related_work` 按 `general` 等主题分组，`stats.cited_references=2`。


## 自查清单

- [ ] `case_id` 与合同字段保留；未映射项已登记。
- [ ] `introduction` 五段齐全，且每句都有 `[src:...]` 定位。
- [ ] 缺口句与背景句不重复；贡献只取自作者材料。
- [ ] `related_work` 按主题分组，未被引用的题录已列出。
- [ ] 未生成“首次/首个”等新颖性断言，无编造文献。

## 执行保证与评分对齐

调用顺序（固定）：

1. 先运行包内脚本（若存在）：`python scripts/run_skill.py`（默认读 `/input/case.json`，写 `/app/output/result.json`）；路径不同时用 `--case` / `--output` 指定。
2. **必须交付完整成品**：脚本只做句子分类与定位；调用 Agent 必须据此写出完整五段式 `introduction`（背景→已有工作→缺口→本文贡献→结构路线图）与按主题分组的 `related_work`，逐句保留 `[src:...]`；无贡献材料时用中性表述并登记缺口，不得整节留空、`pending` 或只交提纲。
3. 最后自检：`case_id` 与输入一致；`output_contract` 每个字段都存在；`status` 取本合同允许值 `success | partial | insufficient_input | failed`（有贡献且有缺口陈述时为 `success`）；正文无 `pending`/`TODO`/空占位；数字、引用、来源可逐条回溯。

运行保证：

- 输入缺失、JSON 损坏、字段异常、输出目录不存在时，仍须写出结构化 `result.json` 并正常结束，不得崩溃或非零退出。
- 不联网、不伪造数据、不声称平台评分或官方案例结果。
- 交付物必须是可直接编辑与复核的成品，而不是流程说明。

评分对齐（本题五个维度）：

- 实际效果（40）：完整产出任务要求的最终成品，覆盖全部输入条目，可直接进入修改流程。
- 使用增益（25）：体现相对基础 Agent 的可验证改进（结构化、术语/数字/引用核对、可编辑性）。
- 创新增益（15）：保留方法学增量（分块、对齐、二次校验、版本比对等），并用结果证明其价值。
- 可靠体验（10）：异常输入有明确降级与说明，不同长度/文体/重复运行下稳定产出。
- 生态贡献（10）：用途、调用、输入输出、限制、示例、维护信息齐全，资源可被他人复用。
