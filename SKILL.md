---
name: koubo-clean
description: 口播视频逐字清理规范：只剪“嘴瓢”（说话卡壳、半截词改口、叠字、紧邻重复、改口残留），保留内容性和教学性重复，字幕与口播同步。适用于已有转录/ChatCut 时间线的口播、讲课、录屏讲解、访谈。已有转录才用本 skill；没有转录先 media-transcribe（本地）或 ChatCut 转录；只要审查清单用 AI剪口播；完整课程成片流程用 dou-ai-course-video。触发词：剪口播、清理口播、逐字清理、去口癖、去叠字、去嘴瓢、口播去重、talking-head cleanup。
---

# 口播清理（koubo-clean）

豆哥的口播剪辑决策规范。适用于任何口播/讲课/录屏讲解/访谈类视频的逐字清理：**只剪“嘴瓢”，不删内容**。

## 何时使用 / 何时不用（Agent 匹配决策）

**使用**：已有转录或 ChatCut 时间线，任务是清理口误/叠字/重复、保持字幕与口播同步。

**不用（交给相邻 skill）**：
- 没有转录，需要先把音视频转成文字 → `media-transcribe`（本地高精度）或 ChatCut 转录
- 只要口误识别与删除任务清单（不实际剪辑）→ `AI剪口播`
- 完整课程成片流程（建项目/字幕样式/贴片/导出）→ `dou-ai-course-video`
- 改素材位置、加 B-roll、改字幕样式等非口误修改 → `chatcut-plugin-basics`

## 适用场景

- 口播课、录屏讲解、访谈、演讲的单人/多人素材
- 在 ChatCut 中完成：`read_script` → 编辑 `timeline.md` → `apply_script` → `edit_captions refresh`

## 启动检查

1. 确认已有转录：`read_script` 能读到成片文本；读不到先转录（见“何时不用”）
2. 需要看停顿就 `showSilence=true`
3. 明确本次范围：只清理口误，不做内容删减/结构改动

## 决策标准

### 该剪（直接剪）

- 半截词改口：一个技，一个技能包 → 一个个技能包；它也是一个插，它也是一个接口 → 它也是一个接口；告诉说，告诉这个 agent → 告诉这个 agent
- 叠字卡壳：操…作 → 操作；prompt 我们的 prompt → prompt；封装好好 → 封装好；操作操作平台 → 操作平台；把把、呃呃、自自己、一一种
- 紧邻同义重复：肯定…肯定；这个…这个；更早的开始，更早的开始；遇到相同的问题，遇到相同的问题；而不是说…而不是说；就完全的自主了 ×2
- 改口残留：那回到我们，；甚至说现在很火的，现在说；像感啊；我们…我们
- 同一层意思紧挨着说两遍：问题本身模糊，就是需求本身模糊 → 二选一保留

### 不剪（保留）

- 教学性/刻意重复：演示性“拆成…拆成…”、递进式“封装成了技能包的形式，封装成了使用说明”
- 用户明确保留：如“能力的边界，模型的边界”“两种表达方式”
- 跨句/跨段的语义重复：先列清单给用户确认，不擅自删
- 拿不准的 ASR 疑点：保留并列入清单让用户听原话

## 工作流

1. 读当前成片：`read_script`（需要看停顿就 `showSilence=true`）
2. 按 `references/patterns.md` 口误模式库扫描成片文本，找候选
3. 强候选直接剪；中候选（完整的同义句）列给用户确认；ASR 疑点进待确认清单（项目根目录 `pending-words.md`）
4. 编辑 `timeline.md`：
   - 行内叠字用 `~~字~~`；跨多行保留一半时注意 `[sN@M]` 标识
   - 删除整行时连带清掉相邻一个 `[silence]` 标记，避免两段停顿叠加
5. `apply_script` 提交（可先 preview）
6. `edit_captions refresh`，字幕与口播同步
7. 用 `find_transcript` / `read_captions` 抽查关键句，确认字幕与口播一致
8. 按 `references/report-template.md` 生成剪辑报告贴给用户，并把待确认项追加到 `pending-words.md`

## 字幕修正原则

- 字幕文本修正优先 `set_card_text`（custom override），不要改源 ASR；改源会让 timeline 行失配
- 品牌词全系列统一（如 workbuddy / Dify / Agnes / Boson / DeepSeek / Seedream / Seedance / K12 / MiMo / 助学群），发现新变体先向用户确认再统一
- `merge_cards` 不能合并不同 source item 的卡；需要拼接时用“前卡尾 + 后卡头”拆文本

## 常见坑

- `apply_script` 校验可能用旧 ASR 缓存：fix 后按旧拼写写行可过校验，缓存刷新后需新拼写
- `find_transcript` 命中源转录，卡 override 不影响搜索命中
- 口误重复与语义重复的区别：口误是“一个技，一个技能包”“它也是一个插，它也是一个接口”这类，剪；语义是“学完 10 节课能做出什么”多处出现这类，列清单等确认，不剪

## 输出

- 每节清理完，向用户汇报：剪了哪些（用实际口播内容描述，不用 `[sN]` 内部编号）、成片时长变化、保留/待确认项
- 待确认项集中到 `pending-words.md`，用户一次性核对后逐条处理

## 完成判定（验收清单）

以下全部满足才算完成：

1. 候选口误全部处理完：直接剪掉的已提交（`apply_script`），中候选已列给用户确认，ASR 疑点已进 `pending-words.md`
2. `edit_captions refresh` 已执行，字幕与口播同步
3. 用 `find_transcript` / `read_captions` 抽查关键句，确认修正生效
4. 剪辑报告已按 `references/report-template.md` 输出给用户
