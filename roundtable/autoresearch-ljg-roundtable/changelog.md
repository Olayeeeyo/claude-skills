# Autoresearch Changelog — ljg-roundtable

## Baseline (Experiment 0)

**Score:** 25/26 (96.2%)
**Runs:** 5 runs across 2 topics (投资理性 × 3, AI方法论 × 2), 1 run with user participation
**Failing evals:**
- EVAL 4 失败 1 次：哈拉维的「简言之」为30字，超过25字上限（技巧：skill 只说"一句话压到极致"，没有给出具体字数上限）
- EVAL 5 只测试了 1 次（用户参与场景），通过，但用户反映实际使用中出现过奉承问题——需要在 skill 中加入显式禁止规则

**已知弱点：**
1. 「简言之」没有字数上限，导致偶发超长
2. 主持人行为准则中没有明确的"禁止奉承用户"条款

---

## Experiment 1 — KEEP

**Score:** 5/5 (100%)
**Change:** 
1. 在第 63 行添加「简言之」字数规则：`严格控制在25字以内，是对该段核心论断的极度提炼，而非发言摘要的改写`
2. 在主持人行为准则（第 167 行）添加禁止奉承规则

**Reasoning:** 针对两个已知失败模式添加显式约束

**Result:** EVAL 4 全部通过（所有「简言之」均在 13-19 字范围内，无超限）

---

## Experiment 2 — KEEP

**Score:** 6/6 (100%)
**Change:** 验证禁止奉承规则是否生效（用户参与场景测试）

**Reasoning:** 用户参与发言时，测试参会者是否做出实质性回应而非夸张褒奖

**Result:** EVAL 5 通过
- 四位参会者对用户发言的回应模式："部分同意前提 + 质疑结论" / "引申概念 + 指出描述不准确" / "承认重要性 + 指出逻辑问题" / "补充操作问题 + 追问框架边界"
- 无"揭示了最深刻的事实"、"非常犀利的观察"、"触及了问题的核心"等奉承语言
- 无参会者突然改变立场支持用户

---

## 最终总结

**Baseline → Final:** 96.2% → 100%
**Experiments run:** 2 (both kept)
**Top changes:**
1. 添加「简言之」≤25 字上限规则
2. 添加主持人/参会者禁止奉承用户规则

**Files modified:**
- `~/.claude/skills/ljg-roundtable/SKILL.md`（2 处修改）

**Remaining issues:** 无
