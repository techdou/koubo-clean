# koubo-clean

口播视频逐字清理规范：只剪“嘴瓢”（说话卡壳、半截词改口、叠字、紧邻重复、改口残留），保留内容性和教学性重复，字幕与口播同步。

适用于已有转录 / ChatCut 时间线的口播、讲课、录屏讲解、访谈类视频。

## 何时使用

- 已有转录或 ChatCut 时间线，任务是清理口误/叠字/重复
- 需要保持字幕与口播同步

### 何时不用（交给相邻 skill）

| 场景 | 使用 |
| --- | --- |
| 没有转录，需要先把音视频转成文字 | `media-transcribe`（本地）或 ChatCut 转录 |
| 只要口误识别与删除任务清单（不实际剪辑） | `AI剪口播` |
| 完整课程成片流程（建项目/字幕样式/贴片/导出） | `dou-ai-course-video` |
| 改素材位置、加 B-roll、改字幕样式等非口误修改 | `chatcut-plugin-basics` |

## 决策标准

**该剪（直接剪）**

- 半截词改口：一个技，一个技能包 → 一个个技能包
- 叠字卡壳：操…作 → 操作；把把、呃呃、自自己、一一种
- 紧邻同义重复：肯定…肯定；这个…这个
- 改口残留：那回到我们；像感啊
- 同一层意思紧挨着说两遍：问题本身模糊，就是需求本身模糊 → 二选一保留

**不剪（保留）**

- 教学性/刻意重复、递进式重复
- 用户明确要求保留的表达
- 跨句/跨段的语义重复（先列清单给用户确认）
- 拿不准的 ASR 疑点（保留并列入待确认清单）

## 工作流

1. `read_script` 读当前成片（需要看停顿就 `showSilence=true`）
2. 按 `references/patterns.md` 口误模式库扫描，找候选
3. 强候选直接剪；中候选列给用户确认；ASR 疑点进 `pending-words.md`
4. 编辑 `timeline.md` → `apply_script` 提交
5. `edit_captions refresh`，字幕与口播同步
6. `find_transcript` / `read_captions` 抽查关键句
7. 按 `references/report-template.md` 输出剪辑报告

## 目录结构

```
koubo-clean/
├── SKILL.md                # 技能主文档（Agent 入口）
├── README.md
└── references/             # 必要说明文档（版本控制内）
    ├── patterns.md         # 口误模式库
    └── report-template.md  # 剪辑报告模板
```

## 依赖

- ChatCut（`read_script` / `apply_script` / `edit_captions` 等）
- 已有转录或 ChatCut 时间线
