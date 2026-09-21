# `_meta` · 题库元数据（知识点目录 + 试题映射）

本目录存放题库的「知识点目录（受控词表）」与「试题 → 知识点映射」，供刷题 App / 脚本 / CI 共用。
**docx 本身无需任何改动**，只在此基础上追加元数据。

## 文件约定

| 文件 | 说明 |
|---|---|
| `robot_knowledge_catalog.json` | 机器人等级考试一~八级知识点目录（受控词表）。`version` 变更表示词表调整 |
| `robot_knowledge_catalog.md` | 同内容可读版，便于人工审阅 |
| `knowledge_map/<paperKey>.json` | 每套试卷的「卷内题号 → 知识点」映射 |

### `paperKey` 规则

`YYYY_MM_robot_<级别>`，如 `2026_06_robot_3`，与 App 内缓存 key 一致：
由 docx 文件名中的年份、月份与末尾的「N级」推导。

### `knowledge_map/<paperKey>.json` 结构（`schema: algorobo.knowledge-map/1`）

```jsonc
{
  "schema": "algorobo.knowledge-map/1",
  "paperKey": "2026_06_robot_3",
  "paper": "2026年6月中国电子学会机器人技术等级考试3级.docx",
  "level": 3,
  "levelName": "三级标准",
  "catalog": "robot_knowledge_catalog.json",
  "catalogVersion": 1,
  "method": "rule-keyword-v2",
  "generated": "2026-09-21",
  "stats": { "questions": 31, "matched": 12, "unmatched": 19 },
  "points": {                       // 卷内题号 -> 知识点（未命中的题号不出现）
    "1": ["简单机械原理 / 杠杆", "省力杠杆与费力杠杆"]
  },
  "questions": [                    // 明细，含题干摘要，便于人工复核
    { "index": 1, "type": "单选题", "hasImage": false,
      "stemHead": "如图自行车中使用了省力杠杆…",
      "knowledgePoints": ["简单机械原理 / 杠杆"], "scores": [6], "matched": true }
  ],
  "unmatched": [                    // 未命中的题（多为图片题，需视觉模型或补词表）
    { "index": 7, "stemHead": "电路原理图如下…" }
  ]
}
```

## 使用方式

- **App 侧**：导入某卷后先读 `knowledge_map/<paperKey>.json`，命中即写入题目知识点（离线、零成本）；
  未收录或未命中的题，再走本地关键词规则 / AI（含视觉）兜底。
- **新增试卷时**：
  1. 把 docx 放到对应年份目录（保持既有命名风格，App 靠 Git Trees API 自动发现）；
  2. （推荐）补一份 `knowledge_map/<paperKey>.json`：可先脚本生成初稿，再人工校对；
  3. 若出现词表里没有的考点，先在 `robot_knowledge_catalog.json` 里补条目（并把 `version` +1），再补映射。

## 当前覆盖情况（2026-09-21 生成）

- 试卷：**57 套**（2024/2025/2026 × 1~6 级）
- 题目：**2145 道**，已匹配知识点 **1151 道（53.7%）**
- 未匹配的主要原因：题干为「如下图 / 电路原理图 / 程序模块如下图」等**图片题**（判别信息在图内），
  以及考级标准词表用词较抽象（如「控制系统工作流程」）而题目用具体词（Arduino/端口/单位符号）。

生成方式：`docx 解析 + 受控词表（按试卷等级限定）+ 领域关键词扩展表` 离线规则匹配（method=`rule-keyword-v2`）。
后续提升方向：① 继续扩充关键词扩展表；② 对 `hasImage=true` 的题改用视觉模型识别；③ 人工校对回填。
