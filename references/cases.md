# 标段录入案例

真实跑通的录入方案与分支判断。取值口径见 `swggzyjy-bd-rules.md`，面板 id 见 `swggzyjy-bd-panels.md`。

---

## 案例 A：暗标测试标段完整录入（汕尾 SWJG2026-0014）

**目标**：为「汕尾市暗标测试房屋建筑工程总承包项目（20261109）」（暗标，房屋建筑工程总承包，
项目总投资 100.00000000 万元，标准工期 300 天）录入标段测试数据并保存落库。

**结果**：标段 id `7ebe5a4968b74ef0b3f86bd22d11636b`，四张评审表各 1 行、招标文件 1 份，
查库与刷新回显全部一致。

### A1. 父级读到的口径

| 项 | 值 | 来源 |
|---|---|---|
| 项目总投资 | 100.00000000 万元 | 父级交易申请「工程信息」面板 |
| 标准工期 | 300 天 | 父级 |
| 交易类型 zbxmlx | A01 房屋建筑 | 父级 |
| 标段类型 zblx | 07 总承包 | 父级 |
| 评定分离 dbms | 03 | 父级 → 评标办法只剩 06/07/08，且出现「定标配置」页签 |

### A2. 逐字段取值

| 字段 | 值 | 依据 |
|---|---|---|
| 标段分类 `bdflCode` | A08 工程施工 | 码表 `jsgcbdfl` |
| 标段名称 `bdmc` | 汕尾市暗标测试房屋建筑工程总承包项目（20261109） | 父级回填 |
| 招标内容 `jsnrhgm` | 109 汉字（见 A3） | ≥50 汉字硬校验 |
| 招标范围及规模 `zbfw` | 122 字（见 A3） | ≥50 汉字硬校验 |
| 标准工期 `bzgqrlt` | 300 | 父级 |
| **是否暗标 `sfab`** | **是** | 本次是暗标测试项目 |
| 资格审查方式 `zgscfsCode` | 02 资格后审 | 常规取值 |
| 投标人类型 `sgrlxCode` | 2 企业 | 常规取值 |
| 标书副本数量 `bsfbsl` | 2 | 常规取值 |
| 合同估算价 `htgsjy` | 980000.00 | 控制价 × 0.98，且必须 ≤ 控制价 |
| 招标控制价 `zbkzjy` | 1000000.00 | = 项目总投资 |
| 保证金 `bzjy` | 10000.00 | 控制价 × 1% |
| 评标办法 `pfff.pbbfCode` | 06 定性评审 | `dbms=03` 可选 06/07/08；06 最省必填项 |
| 评标委员会人数 `pbwyhrs` | 5 | 5~50 的单数 |
| 定标办法 `dbbfCode` | 01 单轮票决法 | 用户指定 |
| 定标委员会人数 `dbwyhrs` | 7 | ≥7 的单数 |

保证金支付方式**不用填**，后端 `saveRootEntity` 会固定写入。

### A3. 两段长文本（都过 50 汉字）

**招标内容（`jsnrhgm`）**：

> 本项目为汕尾市建设工程电子暗标功能测试专用项目，模拟房屋建筑工程总承包招标，建设内容包括测试综合楼及配套设施的设计、施工和相关服务。用于验证进场申请、标段暗标配置、投标文件匿名处理及定性评审流程，全部数据仅供测试使用。

**招标范围及规模（`zbfw`）**：

> 本标段招标范围为汕尾市暗标测试房屋建筑工程总承包（20261109）项目的全部施工内容，包含测试综合楼及配套设施的土建、安装、装饰装修及室外配套工程，含施工图设计配合、施工现场管理、竣工验收配合及相关服务，详见招标内容及工程量清单所列全部内容。

### A4. 四张评审列表各 1 行

| 表 | 序号 | 名称 | 说明 |
|---|---|---|---|
| 投标人资格评审项目 | 1 | 建筑业企业资质及安全生产许可证 | 投标人须具备建筑工程施工总承包三级及以上资质，并具备有效的安全生产许可证，且在近三年内没有重大质量安全事故记录。 |
| 形式评审、响应性评审项目 | 1 | 投标文件递交及签到 | 投标人须在投标截止时间前完成电子投标文件签到及递交，逾期送达或未按要求签到的投标文件将被拒收。 |
| 技术评审项目 | 1 | 技术标评审 | 对投标文件的技术部分进行审查，重点评审施工组织设计、进度计划与技术方案的可行性、针对性，暗标处理后不得出现投标人名称等可识别信息。 |
| 商务评审项目 | 1 | 商务标评审 | 对投标文件的商务部分进行审查，重点评审投标报价、工期承诺、合同条款响应情况及项目管理人员配备，暗标处理后不得出现投标人名称等可识别信息。 |

写法要点：

- 说明文字要**贴合项目实际**（资质等级、递交规则、暗标匿名要求），别写"测试""无"这类空话。
- **每张表只留 1 行**。多余的空行会让数据变脏，也会让人误判"已录入"。
- 长文本走「输入 内容要求」/「输入 评审说明」弹窗写，写完用
  `query(kind="dom", selector="#panelXXXTable tbody")` 确认是**单份**内容。

### A5. 招标文件附件

- 本机文件：`doc/testFile/招标文件.pdf`（293 KB）。
- 平台「生成招标文件」按钮被 `c:if false` 隐藏，只能手动上传。
- **顺序**：先「暂存」拿到标段 id，再上传（上传要求实体已有 id），最后「保存」。
- 上传后文件名被平台自动改成 `汕尾市暗标测试房屋建筑工程总承包项目（20261109）招标文件.pdf`
  （规则 = 标段名称 + "招标文件"），自己不用拼。
- 硬校验：必须**恰好 1 份**（0 份报"请上传招标文件（单份）"，多于 1 份报"招标文件数量为一份"）。

### A6. 保存与验库

```powershell
# 一条 SQL 看六张表
powershell -ExecutionPolicy Bypass -File ..\spl5-browser-test\scripts\dbq.ps1 `
  -ProjectRoot 'D:\zsoft\汕尾公共资源\code-swggzyjy' `
  -Sql "SELECT (SELECT COUNT(*) FROM jsgczbxmbd WHERE id='7ebe5a4968b74ef0b3f86bd22d11636b') bdb, (SELECT COUNT(*) FROM jsgczbtbrzgtj WHERE ssxmbdId='7ebe5a4968b74ef0b3f86bd22d11636b') tbrzgtj, (SELECT COUNT(*) FROM jsgczbtbrfhxsc WHERE ssxmbdId='7ebe5a4968b74ef0b3f86bd22d11636b') fhxsc, (SELECT COUNT(*) FROM jsgczbxmjsyq WHERE ssxmbdId='7ebe5a4968b74ef0b3f86bd22d11636b') jsyqb, (SELECT COUNT(*) FROM jsgczbxmswyq WHERE ssxmbdId='7ebe5a4968b74ef0b3f86bd22d11636b') swyqb, (SELECT COUNT(*) FROM jsgczbxmbdfj WHERE ownerId='7ebe5a4968b74ef0b3f86bd22d11636b' AND category='zbwj') zbwj"
```

实测结果：六项全为 1。

---

## 案例 B：评标办法怎么选（dbms=03 评定分离）

`dbms=03` 时下拉里只剩 06/07/08，前端把 01/02/03/04/05 的 option 隐藏了。

| 办法 | 代码 | 额外必填 | 适合 |
|---|---|---|---|
| 定性评审 | 06 | 技术评审 ≥1 条 + 说明、商务评审 ≥1 条 + 说明 | **暗标匿名评审、测试造数首选** |
| 定量评审-总分制 | 07 | 技术分/商务分/价格分、权重、号球数、技术商务评分表、总价量化表 | 真做评分方案时 |
| 定量评审-合格制 | 08 | 同上另加合格制相关项 | 同上 |

06 分支**不要求** js f/sw f/jgf 分数、号球数、下浮系数、技术商务评分表、总价量化表、多报价配置——
录入成本最低，也最贴暗标场景。

另外：技术评审与商务评审面板**默认被 `initHidden()` 隐藏**，选定评标办法后才显示。
看不到面板先查评标办法选没选。

---

## 案例 C：编辑已有标段 vs 新建

本项目父级 `sfhfbd=0`（不划分标段），父级「新建标段」入口会被 `checkBds` 挡住
（返回 `xmbdsm=1`），只能编辑已有标段。

编辑时的入口选择很关键：

| 入口 | URL | 结论 |
|---|---|---|
| 父级列表行内「编辑」`#action454240462` | `cu/{{entity.id}}?` | **缺参数 → 服务端跳过分支校验 → 假落库** |
| custom.jsp 的 `openBdbj` 入口 | `cu/<id>?zbxmlx=A01&zblx=07&dbms=03` | 正确 |

手动补参数时照平台 `initBdbj` 生成的格式写：
`?zbxmlx=A01&zblx=07&dbms=03`。

**为什么这条这么重要**：缺参数 → `getShowAllTab(null,null)` 返回 false →
"至少录入1个资格条件""评标委员会人数5~50""至少录入1个技术评审项"等**整段校验都不执行**。
数据看起来存好了，实际上一条规则都没跑过。刷新页面时若发现某个列表空着，
第一件事是确认 URL 参数齐不齐，再怀疑别的。

---

## 案例 D：新建标段（实测，2026-11-10）

### D1. 父级处理页的 URL 里的 id 是**流程任务 id**，不是业务单据 id

这是最容易卡死的一步。`jsgcjysqcly/cu/<id>` 的 `<id>` 必须是
**`jsgczbxmsqlcgzx` 表里的行 id**（流程任务），不是 `jsgczbxmsq` 的业务 id。

用业务 id 访问会直接抛：

```
could not find [swggzyjy.domain.Jsgczbxmsqlcgzx] with id [c4fd0873fe3e...]
zsoft.data.DataNotFoundException
```

查库的连法：

```sql
-- 交易申请的业务 id ↔ 流程任务 id
SELECT o.id xmId, o.wmc, o.sfhfbd, o.zbxmlxCode, o.zblxCode, o.dbmsCode,
       t.id taskId, t.activityName, t.handled
FROM jsgczbxmsq o
JOIN jsgczbxmsqlcgzx t ON t.workflowInstanceId = o.id
WHERE t.activityName = '交易项目代理录入'   -- 要能录标段的那个节点
ORDER BY t.createdTime DESC;
```

⚠️ `jsgczbxmsqlcgzx` 是**单表继承**：`dtype` 区分实体，`ownerId` 大多为 null，
**业务 id 在 `workflowInstanceId` 列**。

### D2. `sfhfbd` 决定能不能新建第二个标段

| `sfhfbd` | 父级「新建」按钮行为 |
|---|---|
| `false`（不划分标段） | 调 `checkBds`，若已存在标段则弹「该项目不划分标段，只能存在1个标段」并**阻止** |
| `true`（划分标段） | 不做检查，直接 `POST .../jsgcjysqcly/zc` 后打开标段创建页 |

所以**要新建第 2 个标段，必须挑 `sfhfbd=1` 的父级项目**。
只有 `sfhfbd=0` 的项目 → 只能编辑已有标段。

### D3. 新建标段页的 URL 形态

从父级点「新建」会 `z.openWindow` 打开**新窗口**（不是标签页），URL：

```
/jyzxt/jsgcjy/gkzb/jysq/jsgczbxmbdbjcly/cu
  ?jsgczbxmsq=<交易申请业务id>&zbxmlx=A01&zblx=01&sfhfbd=true&dbms=01
```

（编辑已有标段则是 `cu/<标段id>?zbxmlx=..&zblx=..&dbms=..`）

若 `z.openWindow` 的窗口不方便操作，可以**手工拼上面这个 URL 用 `open_tab` 打开**，
效果等价（父级暂存已由 `openBdbj` 完成）。

新建页**字段全空、不预填**（`initData` 已被注释）：
标段分类 9 个 checkbox 全空、资格审查方式 `--请选择--`、两个长文本空、
投标人类型默认「企业」已勾。**必须手工点「新增」建评审行**。

### D4. `dbms` 决定页签与评标办法分支

`dbms=01` 时**没有「定标配置」页签**，评标办法下拉是完整的 01~08；
`dbms=03`（评定分离）才有「定标配置」页签，且评标办法只剩 06/07/08。
**新项目要先确认自己的 `dbms`，案例 A 的结论不能直接套。**

### D5. 暂存 / 保存都要过确认框

点「暂存」弹 dialog-plus 确认框「是否暂存」，**必须点确认框里的「是」**
（`button.confirm`）才会真正提交。不点 → 表单数据全丢、库里没有记录，
但页面上看起来"点过了"。

「保存」同理，弹「是否保存项目标段？」。

### D6. 金额字段要特殊手法（`numberFormat`）

见 `spl5-browser-test` skill 的 `browser-tool-tactics.md`：
`fill`/`type` 在 `.numberFormat()` 字段上会**追加**而不是覆盖，
且 `query(kind="dom")` 读到的 `value` 是 HTML 属性、**不代表实际值**。

---

## 案例 E：新建标段 + 保存全闭环跑通（2026-11-10 实测）

**目标**：在父级 `82a8a4464f484ac8b7bb182977a65122`（SWJG2025-0018 / A01 房建 /
`sfhfbd=1` / `dbms=01`）下新建标段，走完「暂存 → 上传附件 → 录评审表 → 保存 → 刷新 → 查库」。

**结果**：标段 id `0a6886bae18440f086842419b2475c13`，
**七张表全部核对通过**，刷新回显一致。

### E1. `dbms=01` + 04 评标办法的必填矩阵（最小可跑组合）

`dbms=01` 时评标办法下拉是 01~08 全部可选，选 **04 经评审的最低投标价法**（`pbbfCode='04'`）
必填项最少：

| 项 | 取值 | 说明 |
|---|---|---|
| 评标办法 | 04 经评审的最低投标价法 | 选完自动预填 技术分0 / 商务分0 / 价格分100 |
| 价格类型 | 总价（`pfff.bjlxCode`） | 平台预填 |
| **是否限制评审数量** | **否** | 选「否」可跳过号球数/下浮系数/进入评审数量，见 E2 |
| 评标委员会人数 | 5 | `pbwyhrs`，5~50 的单数 |
| 投标总价量化表 | **必须 ≥1 行** | 04 分支的硬校验（`JsgcbdbjService` L937-940） |
| 技术评审 / 商务评审表 | **不需要** | 只在 `pbbfCode=='06'` 时 show |

**技术评审/商务评审面板只在 `pbbfCode=="06"` 时显示**（custom.jsp L770-772），
01/02/04 全部隐藏。所以 04 分支只需录 3 张表：资格评审、形式评审、投标总价量化表。

### E2. 「是否限制评审数量」选「是」会多出 4 个必填项

选 04 评标办法后，平台默认把「是否限制评审数量」置**是**，此时点保存会报：

```
进入评审数量必须大于等于3；请填写抽取下浮系数低值；请填写抽取下浮系数高值；请正确填写号球数
```

源码依据（`JsgcbdbjService.java`）：
- L925-936：`sfxzpssl != false` 时 → 下浮系数低值/高值/号球数必填；
- L399：`sfxzpssl == true` 时 → 「进入评审数量」必须 ≥3。

**房建市政 + 04 评标办法允许不限制评审数量**（代码注释 L924 明写）。
所以点「否」即可一次省掉 4 个字段：

```sql
-- radio 的 id 每次生成都可能变，按 name+value 找更稳
SELECT ... FROM jsgczbxmpfff WHERE ... ;  -- 确认 sfxzpssl=0
```
页面定位：`input[name='pfff.sfxzpssl'][value='false']`（本轮 id 是 `#tag1794275687692False`）。

### E3. 投标总价量化表：4 个字段各有一套手法

面板 `panel09ecdg7hjee2`，新增按钮 `#action09ecdg8972hb`，
行内控件 `IE-zjlhb-<row>-{xh,lhys,lhbz,zgtjw}`：

| 字段 | 类型 | 手法 |
|---|---|---|
| `xh` 序号 | `numberFormat` 整数 | click → Ctrl+A → Backspace → type |
| `lhys` 量化因素 | 普通 input（maxlength 200） | `fill` 直接写 |
| `lhbz` 量化标准 | **readonly textarea**（maxlength 2000） | 点它开弹窗 → 填 `.ui-dialog textarea` → 点弹窗「确定」 |
| `zgtjw` 最高调减(万) | `numberFormat` 小数 | click → Ctrl+A → Backspace → type → 点别处 blur |

弹窗「确定」按钮的坑见 `swggzyjy-bd-panels.md`（要用 `query(kind="semantic")` 拿 ref 再点）。

### E4. 暂存 vs 保存：子表只在「保存」时落库

实测顺序：

1. 「暂存」→ 确认框点「是」→ URL 自动从 `cu` 变成 `cu/0a6886ba…`（拿到 id）
   → 查库：主表在、`sfzc=1`，**三张子表全部 0 行**。
2. 上传招标文件附件 → `jsgczbxmbdfj` 立刻有 1 行。
3. 点「保存」→ 确认框点「是」→ **子表一并落库**，`sfzc=0`。
4. 刷新页面：四个页签的回显与库里一致。

⚠️ 刷新后**评标办法会被重置为 `--请选择--`**（如果只暂存没保存）。
这是正常的 —— 暂存只提交主表。查评标办法要 `JOIN jsgczbxmpfff p ON p.id=b.pfffId`。

### E5. 七表核对 SQL（最终结果全为 1）

```powershell
powershell -ExecutionPolicy Bypass -File ..\spl5-browser-test\scripts\dbq.ps1 `
  -ProjectRoot 'D:\zsoft\汕尾公共资源\code-swggzyjy' `
  -Sql "SELECT (SELECT COUNT(*) FROM jsgczbxmbd WHERE id='0a6886bae18440f086842419b2475c13' AND deleted=0) bdb, (SELECT COUNT(*) FROM jsgczbtbrzgtj WHERE ssxmbdId='0a6886bae18440f086842419b2475c13') tbrzgtj, (SELECT COUNT(*) FROM jsgczbtbrfhxsc WHERE ssxmbdId='0a6886bae18440f086842419b2475c13') fhxsc, (SELECT COUNT(*) FROM jsgczbxmtbzjlhb WHERE ssxmbdId='0a6886bae18440f086842419b2475c13') zjlhb, (SELECT COUNT(*) FROM jsgczbxmbdfj WHERE ownerId='0a6886bae18440f086842419b2475c13' AND category='zbwj') zbwj"
```

主表实测值：`bdflCode=A08`、`bzgqrlt=111`、`zbkzjy=1000000.00`、`htgsjy=980000.00`、
`bzjy=10000.00`、`bsfbsl=2`、`sfab=1`、`sfzc=0`、`zgscfsCode=02`、`sgrlxCode=2`、
`bjlxCode=01`、`pbwyhrs=5`、`deleted=0`。

### E6. 附件上传控件在 iframe 里

「上传」按钮（招标文件行）打开弹窗，弹窗里是 `input[type=file]`，
但它在 **iframe** 中（`browser-frame:xxx`），主文档查不到：
`query(kind="dom", selector="input[type=file]")` 报 "Element not found"。

正确姿势：`inspect` → 找到 iframe 内那个 `input` 的 ref → `upload_files(ref, paths)`。

上传成功后平台自动把文件名改成 `标段名称 + 招标文件.pdf`，本轮得到
`汕尾市技能验证测试房屋建筑工程施工标段（20261110）招标文件.pdf`（299651 字节）。

### E7. 502 与「跳登录页」要分开判别（注意：502 往往是**发布重启**，不是崩溃）

本轮中途上传附件时整个站点返回 **502 Bad Gateway** 持续约 4 分钟。

**用户确认的真相：这是有新功能发布、触发应用重启，不是系统崩溃。**
测试环境会不定期发版，发布窗口内应用进程重启，nginx 短暂拿不到后端 → 502。
**排查时不要往"崩溃/OOM/链路故障"方向猜**，先假定是发布重启。

本轮的实测佐证（SSH 到 192.168.40.13）：

- 应用进程 `/zsoft/app/swggzyjy_jar/swggzyjy.jar`（PID 3738955，监听 **8091**）**一直是活的**，
  `docker ps` 里也没有它的容器（它跑在宿主机上，不在容器里）。
- 直接 `curl http://127.0.0.1:8091/jyzxt/` 返回 **302**（跳登录）→ 应用本身没问题。
- 域名 `test.swggzyjy.rongzhiwang.com` 解析到 **125.88.9.130**（外部网关），
  不是这台机器 → 502 出现在**网关到应用之间**。
- 日志里满屏 `ConnectTimeoutException: Connect to 19.96.245.89:8089`（中台）/
  `19.15.0.128:8080`（粤政易）—— 定时任务连不上外网接口在刷超时，**是噪声不是根因**，
  别把它当成 502 的原因。

**处置顺序**：

1. 先分清是 502 还是跳 `/login`（两种完全不同，见下）。
2. 是 502 → **不要下任何业务结论**（页面根本没渲染，"找不到控件"是假象），
   等 1~2 分钟重试；期间已提交的数据**不会丢**。
3. 恢复后如果被 302 到 `/login`（账号框预填「代理机构1」）→ **session 过期**，
   请用户手动重新登录，**不要代填密码/验证码**。
4. 恢复后从中断处接着做，本轮实测暂存的数据完好、可以直接续做上传和保存。

