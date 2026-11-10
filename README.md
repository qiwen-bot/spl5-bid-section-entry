# spl5-bid-section-entry

[![GitHub](https://img.shields.io/badge/GitHub-qiwen--bot/spl5--bid--section--entry-181717?logo=github)](https://github.com/qiwen-bot/spl5-bid-section-entry)

用内置浏览器在 **SPL5 建设工程平台**（汕尾 / 云浮 / 韶关等）快速录入
「建设工程招标项目标段」测试数据，并确认真的落库。

An Agent Skill for entering construction-engineering **bid section** test data on the
SPL5 public-resource trading platform and verifying that it reaches the database.

> **仓库地址**：<https://github.com/qiwen-bot/spl5-bid-section-entry>

标段是建设工程交易申请下面的一级子单据：字段多、必填规则硬、校验全在后端 service 里。
这份 skill 把「要填什么、填什么值、按什么顺序保存、怎么确认存上了」固化下来。

> **前置依赖**：先安装 [`spl5-browser-test`](https://github.com/qiwen-bot/spl5-browser-test)
> ——SPL5 页面骨架、浏览器工具行为、查库方法都在那里。本 skill 只讲标段业务。

---

## 它解决什么问题

标段录一次数据要过的坎：

- **必填项随评标办法变**：`06 定性评审` 要技术+商务评审；`04 最低投标价法` 反过来要
  ≥1 行投标总价量化表，但**不要**技术/商务评审
- **URL 少一个参数 → 整段校验被跳过**：手工拼 URL 或点列表行内「编辑」，
  服务端 `getShowAllTab(null,null)` 返回 false，"至少录入1个资格条件"等规则**一条都不跑**
- **表名看串**：`jsgczbxmbazl` **不是**投标总价量化表（真实表是 `jsgczbxmtbzjlhb`）
- **「暂存」和「保存」落库范围不同**：暂存只提交主表，子表全空 —— 这不是 bug
- **父级 URL 里的 id 是流程任务 id**，不是业务单据 id，用错直接抛 `DataNotFoundException`
- **`bit(1)` 列（`sfab`/`sfzc`/`deleted`）套 `CONCAT` 会读出空白**，误判成"没保存"

---

## 安装

### 方式一：直接 clone

```bash
git clone https://github.com/qiwen-bot/spl5-bid-section-entry.git
```

### 方式二：手动放置

把整个 `spl5-bid-section-entry/` 放进你的 agent 的 skills 目录：

| Agent | 路径 |
|---|---|
| MiniMax Code / Mavis | `~/.minimax/agents/<agent-name>/skills/spl5-bid-section-entry/` |
| Claude Code | `~/.claude/skills/spl5-bid-section-entry/` |

其他客户端的 skills 目录位置各不相同，参见各自文档。

无第三方依赖。

---

## 用法

skill 由 agent 按 `SKILL.md` 里的 Procedure 自动执行。典型触发语：

> "在汕尾测试环境新建一个建设工程标段，把资格评审、形式评审和招标文件录进去，保存后确认落库。"

需要用户拍板的两件事（会实质改变要录的内容）：

1. **评标办法** —— `dbms=03` 评定分离时只剩 06/07/08；`dbms=01` 时 01~08 全可选。
   **`04` 必填项最少**，适合快速造数。
2. **定标办法** —— 01 单轮票决法 / 02 多轮票决法 / 03 集体议事法（仅 `dbms=03` 有此页签）

---

## 内容

| 文件 | 内容 |
|---|---|
| [`SKILL.md`](SKILL.md) | 主流程：定位页面 → 读父级口径 → 选评标办法 → 录主表 → 录评审表 → 上传附件 → 保存 → 核对 |
| [`references/swggzyjy-bd-rules.md`](references/swggzyjy-bd-rules.md) | 校验规则与取值口径（控制器 + service 两处硬校验） |
| [`references/swggzyjy-bd-panels.md`](references/swggzyjy-bd-panels.md) | 面板与控件对照（汕尾）：页签、面板 id、字段 name、custom.jsp 联动 |
| [`references/verify-bid-section.md`](references/verify-bid-section.md) | 落库核对：七表 SQL、表名看串对照、04 必填矩阵 |
| [`references/cases.md`](references/cases.md) | 真实案例 A~E：完整录入闭环、入口选择、暂存 vs 保存、502 判别 |

在线浏览：<https://github.com/qiwen-bot/spl5-bid-section-entry/tree/main>

---

## 已实测通过的完整组合

汕尾测试环境 `dbms=01` + `04` 经评审的最低投标价法，七表核对全部为 1：

```
bdb | tbrzgtj | fhxsc | zjlhb | zbwj | pfff
1   | 1        | 1      | 1     | 1    | 1
```

最小可跑组合：`04` 评标办法 + 「是否限制评审数量 = 否」+ 评标委员会 5 人
+ 资格评审 1 行 + 形式评审 1 行 + 投标总价量化表 1 行 + 招标文件 1 份。

---

## License

MIT — 见 [LICENSE](LICENSE)。
