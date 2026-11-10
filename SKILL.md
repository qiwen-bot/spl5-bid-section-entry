---
name: spl5-bid-section-entry
description: |
  用内置浏览器在 SPL5 建设工程平台（汕尾/云浮/韶关等）快速录入「建设工程招标项目标段」
  测试数据并确认落库。触发场景：用户要求"录入/填报/造数"标段信息、标段详细评审、
  资格评审/形式评审/技术评审/商务评审项目、招标文件附件，或问"这个标段页要填哪些字段、
  填什么值才过校验、评标办法/定标办法怎么选、保存顺序是什么"。也用于暗标项目
  （是否暗标=是）的标段录入。
  不用于通用 SPL5 页面探索与控件定位（用 spl5-browser-test），不用于模型/代码/DDL
  工作（用 spl5-model-builder / spl5-code-generator / spl5-db-engineer）。
---

# 建设工程标段测试数据录入

标段是建设工程交易申请下面的一级子单据，字段多、必填规则硬、校验在后端 service 里。
本 skill 把"要填什么、填什么值、按什么顺序保存、怎么确认存上了"固化下来。

先读 `spl5-browser-test` skill 拿到 SPL5 页面骨架、浏览器工具行为和查库方法，
本 skill 只讲标段业务。

## Inputs to collect

- **父级交易申请**：URL 或 id（汕尾为 `/jyzxt/jsgcjy/gkzb/jysq/jsgcjysqcly/cu/<申请id>`，
  生成页面 `jsgcjysqcly.jsp`，对外常称"建设工程交易申请处理页"）。
  标段 id 从父级页面的「项目标段」列表进入后拿到，形如
  `/jyzxt/jsgcjy/gkzb/jysq/jsgczbxmbdbjcly/cu/<标段id>`。
- **业务口径**：项目总投资、标准工期、交易类型（zbxmlx，A01 房屋建筑…）、标段类型（zblx，
  07 总承包 / 03 施工…）、评定分离 `dbms`（01 或 03）。
  **必须从父级交易申请页读**，不要用用户口述值直接填，也不要凭空编价格。
- **需要用户拍板的两件事**（会实质改变要录的内容，先问）：
  1. 评标办法：按 `dbms` 决定可选范围 —— `dbms=03` 评定分离时只剩 06/07/08；
     `dbms=01` 时 01~08 全可选。**04 经评审的最低投标价法必填项最少**，适合快速造数。
  2. 定标办法：01 单轮票决法 / 02 多轮票决法 / 03 集体议事法（仅 `dbms=03` 有此页签）
- **招标文件附件**：本机文件路径（平台「生成招标文件」按钮通常被 `c:if false` 隐藏，只能手动上传）。
- 标段是否暗标（暗标测试项目通常 `是否暗标=是`）。

## Procedure

### 1. 定位页面，并从父级点进去

在目标项目里 grep 生成的 JSP 拿面板与按钮 id（**不要照抄别项目的 id**）：

```
src/main/webapp/WEB-INF/views/**/jsgczbxmbdbjcly.jsp
```

汕尾的页签 / 面板 / 按钮对照见 `references/swggzyjy-bd-panels.md`。

**父级处理页的 URL id 是「流程任务 id」，不是业务单据 id**（最易卡死的一步）：

- 交易申请业务 id = `jsgczbxmsq.id`；**流程任务 id = `jsgczbxmsqlcgzx.id`**，
  通过 `jsgczbxmsqlcgzx.workflowInstanceId = jsgczbxmsq.id` 关联。
- 用业务 id 访问 `jsgcjysqcly/cu/<业务id>` 会直接抛
  `could not find [Jsgczbxmsqlcgzx] with id [...]`。
- 查法见 `references/cases.md` 案例 D1。

**必须从父级「交易申请」页面点进标段编辑页**，不要手工拼 URL。父级用

```js
z.openWindow('.../jsgczbxmbdbjcly/cu/' + id + '?zbxmlx=' + zbxmlx + '&zblx=' + zblx + '&dbms=' + dbms)
```

打开，后端 `doValidate` 正是从 URL query string 取 `zbxmlx` / `zblx` / `dbms` 来决定
走哪段校验。手工拼的 URL 少了这些参数 → 服务端**跳过按项目类型分支的校验** →
"保存成功"但校验没跑（假落库）。

⚠️ 父级**列表行内的「编辑」链接**（`#action454240462`）URL 是 `cu/{{entity.id}}?`，**不带参数**。
用 custom.jsp 的 `openBdbj` 入口，或手动补成 `cu/<id>?zbxmlx=A01&zblx=07&dbms=03`。

### 2. 从父级交易申请取口径

打开父级申请页，读出：项目总投资、标准工期、交易类型 `zbxmlx`、标段类型 `zblx`、
评定分离 `dbms`、是否划分标段。价格口径由总投资推：
招标控制价取总投资，**合同估算价必须 ≤ 招标控制价**（后端硬校验），保证金取个整数比例。

### 3. 定值（先算再填）

按 `references/swggzyjy-bd-rules.md` 的约束算出一组**必定过校验**的值：

- 标准工期 = 父级标准工期
- 招标控制价 = 项目总投资；合同估算价 = 控制价 × 0.9~0.99；保证金 = 控制价 × 1%
- 评标办法 = **04 经评审的最低投标价法**（`dbms=01` 时可选，必填项最少：
  技术分/商务分/价格分自动预填 0/0/100，只需 1 行投标总价量化表）
  或 06 定性评审（`dbms=03` 暗标场景，需技术评审 + 商务评审各 ≥1 条）
- **「是否限制评审数量」选「否」**（04 分支）：选「是」会额外要求
  号球数 / 抽取下浮系数低值 / 高值 / 进入评审数量≥3 四个字段（`sfxzpssl`）。
- 评标委员会人数 = 5（要求 5~50 的单数）
- 定标办法 = 01 单轮票决法；定标委员会人数 = 7（要求 ≥7 的单数，仅 `dbms=03`）
- 标书副本数量 = 2；投标人类型 = 企业；资格审查方式 = 02 资格后审
- 标段分类 = 工程施工（A08）

> 评标办法 04/06 分支的完整必填矩阵见 `references/verify-bid-section.md`
> 与 `references/cases.md` 案例 E。

### 4. 录主字段（标段信息 + 标段价格）

- 标段名称 / 地址 / 招标内容通常由父级自动回填，别覆盖成手写值。
- **`招标内容`（列 `jsnrhgm`）和 `招标范围及规模`（列 `zbfw`）是两个独立长文本**，
  各自**至少 50 个汉字**（后端按汉字计数，不是字符数——`outPutChineseCount`）。
  父级回填的值常常不满足，必须按实际工程内容补足。
  注意模型里的 nameCN（"建设内容和规模"）和页面标签（"招标内容"）不一致，**以页面标签和报错文案为准**。
- 价格字段（合同估算价 / 招标控制价 / 保证金）注意 `numberFormat` 会显示千分位。
  ⚠️ **别用 `fill`/`type` 直接写**：`numberFormat` 绑定 `focus → select()`，
  会把新值**追加**到旧值后面（填 `1000000` 变成 `10000001000000`）。
  正确姿势：`click` 聚焦 → `press_key{key:"a",modifiers:["Control"]}` 全选 →
  `press_key{key:"Backspace"}` 清空 → `type` 新值 → 点别处 blur 触发格式化。
  ⚠️ **读值只有 `screenshot` 可靠**：`query(kind="dom")` 和 `query(kind="editable")`
  读到的 `value` 都是 HTML 属性、**不代表实际值**（插件写的是 property），
  会误判"没填进去"。`query(kind="dom")` 对 textarea 读不到内容。
  详见 `references/swggzyjy-bd-panels.md` 的「浏览器操作要点」一节；
  通用版在 `spl5-browser-test` skill 的 `browser-tool-tactics.md`。
### 5. 录评审列表（**按评标办法分支，不是固定四张**）

**哪几张表要录，取决于评标办法**（技术评审/商务评审面板只在 `pbbfCode=="06"` 时 show）：

| 评标办法 | 必录的表 | 页签 |
|---|---|---|
| 04 经评审的最低投标价法 | 资格评审、形式评审、**投标总价量化表** | 资格评审 / 形式评审、响应性评审 / 详细评审 |
| 06 定性评审 | 资格评审、形式评审、**技术评审**、**商务评审** | 同上 + 详细评审 |
| 01/02/07/08 | 另需技术/商务评分表、总价量化表、多报价配置等 | 详细评审 |

结构相同的评审表（序号 / 名称 / 长文本说明），各录 **1 行**：

| 表 | 字段 | 真实表 |
|---|---|---|
| 投标人资格评审项目 | 序号、项目名称(xmmc)、内容要求(nryq) | `jsgczbtbrzgtj` |
| 形式评审、响应性评审项目 | 序号、项目名称(xmmc)、内容要求(nryq) | `jsgczbtbrfhxsc` |
| 技术评审项目 | 序号、评审项目名称(psxmmc)、评审说明(pssm) | `jsgczbxmjsyq` |
| 商务评审项目 | 序号、评审项目名称(psxmmc)、评审说明(pssm) | `jsgczbxmswyq` |
| **投标总价量化表** | 序号(xh)、量化因素(lhys)、量化标准(lhbz)、最高调减(万)(zgtjw) | **`jsgczbxmtbzjlhb`** |

录入要点：

- 每张表点「新增」按钮加行，**只保留一行**（多余的空行会让校验或数据变脏）。
- ⚠️ 行内控件名里的 `<row>` 是**整页全局递增**的，不是每张表从 1 开始
  （实测同页依次新增资格→形式→量化表，拿到的是 `tbrzgtj-3` / `fhxsc-4` / `zjlhb-5`）。
  先 `query(kind="dom", selector="input[name^='IE-zjlhb-']")` 看实际 row 号再填。
- 内容要求 / 评审说明 / 量化标准是 **readonly textarea**，必须点开弹窗写。
- 弹窗是**预填 + 整体覆盖**：打开时会把当前内容带出来，你在光标处续写会把旧值和新字拼在一起。
  正确做法是**全选后重写**，写完用 `screenshot` 看渲染文本（`query(kind="dom")` 读不到
  textarea 内容，见 `references/swggzyjy-bd-panels.md`）。
- ⚠️ **弹窗的「确定」按钮要用 `query(kind="semantic", text="确定")` 拿 ref 再点** ——
  `.ui-dialog` 选择器匹配不到它，`click` 上去不生效。
- 文本内容要贴合项目实际（资质等级、签到递交规则、暗标匿名要求），别写"测试""无"这类空话。

### 6. 招标文件附件

- 平台「生成招标文件」接口要求标段已保存且按钮被隐藏，只能手动上传。
- 上传控件在 iframe 里，`upload_files` 的 ref 取 iframe 内的 file input。
- 文件名规则：**标段名称 + "招标文件"**。平台会自动改名，不要自己拼。
- 必须恰好 1 份。

### 7. 保存顺序（不能颠倒）

```
暂存（拿到标段 id）→ 上传招标文件附件 → 保存（确认框点「是」）
```

原因：附件上传要求所属实体已有 id；先上传会因为没有 id 而失败。

**「暂存」和「保存」都会弹确认框，必须点确认框里的「是」**
（`button.confirm`）才会真正提交。不点 → 表单数据全丢、库里没有记录，
但页面上看起来"点过了"。**点完一定要查库确认**，别只看页面。

**「暂存」只提交主表，「保存」才把子表一起提交**（2026-11-10 实测）：

| 动作 | 主表 | 子表 | `sfzc` |
|---|---|---|---|
| 暂存 | 落库 | **不落库** | 1 |
| 保存 | 落库 | **一并落库** | 0 |

⚠️ 暂存后刷新页面，子表全空、**评标办法会被重置为 `--请选择--`** —— 这是正常的。
查评标办法要 `JOIN jsgczbxmpfff p ON p.id=b.pfffId`，不能只看主表。

底部「暂存/保存/关闭」是裸 `<button>`（无稳定 id），`query(kind="semantic", text="保存")`
会误匹配到表单 label。**用 `screenshot` 看按钮条像素位置 → `click(input={position:{x,y}})`**。

### 7.1 新建 vs 编辑（`sfhfbd` 决定能不能新建第 2 个标段）

| 父级 `sfhfbd` | 父级「新建」按钮行为 |
|---|---|
| `false`（不划分标段） | 调 `checkBds`，已有标段则弹「该项目不划分标段，只能存在1个标段」并**阻止** |
| `true`（划分标段） | 不检查，直接 `POST .../jsgcjysqcly/zc` 后打开标段创建页 |

**要新建标段，先挑 `sfhfbd=1` 的父级项目**；只有 `sfhfbd=0` 的项目只能编辑已有标段。
新建页 URL 形态、字段全空不预填、`dbms` 对页签的影响，见 `references/cases.md` 案例 D。

### 8. 确认落库

刷新页面让服务端重渲染，核对各页签有没有行、码表下拉选中没有、radio checked 没有、
附件条数对不对；**再查一次库**（连接方式见 `spl5-browser-test` skill，
本单据专用的 SQL 见 `references/verify-bid-section.md`）：

```
jsgczbxmbd            1 行：sfab=1、sfzc=0（非暂存）、deleted=0
jsgczbtbrzgtj         1 行：ssxmbdId=标段id
jsgczbtbrfhxsc        1 行：ssxmbdId=标段id
jsgczbxmjsyq          1 行：ssxmbdId=标段id   ← 不是 jsgczbxmtbjsyqb（06 分支才有）
jsgczbxmswyq          1 行：ssxmbdId=标段id   ← 不是 jsgczbxmtbswyqb（06 分支才有）
jsgczbxmtbzjlhb       1 行：ssxmbdId=标段id   ← 投标总价量化表（04 分支才有；不是 jsgczbxmbazl）
jsgczbxmbdfj          1 条：ownerId=标段id 且 category='zbwj'
jsgczbxmpfff          1 行：id = 主表.pfffId，pbbfCode / sfxzpssl / pbwyhrs 正确
```

⚠️ **表名最容易看串**（查错表会得出「没落库」的假结论）：
`jsgczbxmbazl` **不是**投标总价量化表（该表在本库为空）；`jsgczbxmbdfj` 没有 `deleted` 列。

保存后控制台通常报 `TypeError: Cannot read properties of null (reading 'panelXXXXXXX')`
（页面独立打开没有 opener 时的前端缺陷），**不代表保存失败**，以刷新后的回显 + 查库为准。

## Output contract

- 标段 id、所属父级交易申请 id
- 逐字段取值表（字段 → 值 → 口径来源）
- 各评审列表录了什么（表名 / 序号 / 名称 / 说明全文）
- 附件：文件名、大小、上传时间
- 查库核对结果（表名 + 行数 + 关键字段值）
- 遗留项与平台缺陷，并区分"录入没做好"和"平台 bug"

## Failure handling

- **保存没报任何错，但该校验的没校验**（例如该报"至少录入1个资格条件"却没报）：
  多半是手工拼了 URL，缺 `zbxmlx` / `zblx` / `dbms` → 服务端跳过了分支校验。
  从父级「交易申请」页面重新点进去，不要在原地改 URL 重试。
- **校验报错**：按后端返回的原文改。控制器硬校验看 `Jsgczbxmbdbjcly.java#doValidate`，
  分支规则看 `JsgcbdbjService#validateNormal/validateFJSZ`，不要只看前端红字猜。
  报错横幅在页面顶部，**改完不会自动消失**，以文字是否变化或查库为准。
- **"进入评审数量必须大于等于3；请填写抽取下浮系数低值/高值；请正确填写号球数"**：
  04 评标办法 + 「是否限制评审数量=是」导致。把该 radio 改成**否**即可全部跳过
  （房建市政 + 04 明确允许）。
- **"至少录入1个投标总价量化表"**：04 分支的硬校验，量化表要手工点「新增」并录 1 行。
- **"至少录入1个资格条件"**：资格评审表没行——多半是「新增」没点到位或行被误删。
- **"评标委员会人数为5~50内的单数"**：填了偶数或越界。
- **"定标委员会人数为大于等于7的单数"**：定标人数要 ≥7 且奇数（仅 `dbms=03`）。
- **保存后列表没行但库里查不到**：多半查错表（见第 8 步的表对照）或关联列用错。
  `jsgczbxmbazl` ≠ 投标总价量化表；`jsgczbxmbdfj` 无 `deleted` 列。
- **长文本内容重复**：弹窗预填后没清干净导致叠加。用弹窗全选覆盖写一次，并查库确认。
- **弹窗的「确定」点了没反应**：`.ui-dialog` 选择器匹配不到它。
  用 `query(kind="semantic", text="确定")` 拿带 frame 的 ref 再 `click(ref)`。
- **页面突然变成 502 或跳到 /login**：**这两种是不同故障，必须分开判别**。
  - 302 跳 `/login`（账号框预填「代理机构1」）→ session 过期，**请用户手动重新登录**，
    不要试图自己登录。
  - 502 Bad Gateway → **多半是测试环境在发版重启**（不是崩溃）。**不要往"OOM/链路故障"方向猜**，
    也**不要下任何业务结论**（页面根本没渲染，"找不到控件"是假象）—— 等 1~2 分钟重试。
    排查顺序：`Invoke-WebRequest` 探站点 → SSH 到应用机 `curl 127.0.0.1:<port>` 看应用本身
    → 应用本身返回 302 就说明**应用是好的**，问题在网关到应用这一跳。
    恢复后**已暂存的数据不会丢**，可以接着做。
  详见 `references/cases.md` 案例 E7。
- **需要重存**：先查库确认当前库里的值，别拿页面上半成品覆盖库里已经正确的数据。

## Examples

完整案例见 `references/cases.md`：
- 案例 A：暗标标段完整录入（`dbms=03` / 06 定性评审 / 四张表）
- 案例 B：评标办法怎么选
- 案例 C：编辑已有标段 vs 新建、入口 URL 参数坑
- 案例 D：新建标段（流程任务 id、`sfhfbd`、`dbms` 影响、确认框）
- 案例 E：**新建标段 + 保存全闭环跑通**（`dbms=01` / 04 评标办法 / 投标总价量化表 /
  七表核对 SQL / 暂存与保存的差异 / 502 与跳登录的判别）

Input: "帮我在汕尾测试环境为暗标测试项目 SWJG2026-0014 录入标段信息并保存。"

Output 要点：

1. 父级页读口径：总投资 100.00000000 万元、标准工期 300 天、zbxmlx=A01 房屋建筑、
   zblx=07 总承包、`dbms=03` 评定分离。
2. 定值：控制价 1,000,000.00 / 估算价 980,000.00 / 保证金 10,000.00 / 工期 300 /
   评标办法 06 / 评标委员会 5 / 定标办法 01 单轮票决法 / 定标委员会 7 / 副本 2 / 暗标=是；
   招标内容与招标范围及规模各写 ≥50 汉字。
3. 四张评审表各 1 行，内容贴合"暗标 + 总承包 + 三级资质"场景。
4. 暂存 → 上传 `doc/testFile/招标文件.pdf` → 保存 → 确认框点「是」。
5. 查库核对 6 张表，刷新页面复核回显。

## Windows (win32) platform notes

- 查库用 `spl5-browser-test` skill 的 `scripts/dbq.ps1`（PowerShell），需要 JDK 21。
- 仓库路径含中文与空格时，PowerShell 参数一律加引号。
