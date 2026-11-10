# 汕尾 建设工程招标项目标段编辑页 — 面板与控件对照

来源：`swggzyjy2022/src/main/webapp/WEB-INF/views/jyzxt/jsgcjy/gkzb/jysq/jsgczbxmbdbjcly.jsp`
（生成文件，**不要手改**；改逻辑走同名 `.custom.jsp`）。

> 下列 id 只在汕尾这套生成结果里有效。换项目（云浮/韶关）必须重新 grep 同一个 JSP。

## URL

| 单据 | 路径 |
|---|---|
| 交易申请（父级） | `/jyzxt/jsgcjy/gkzb/jysq/**jsgcjysqcly**/cu/<申请id>`（生成页 `jsgcjysqcly.jsp`，对外常称"建设工程交易申请处理页"） |
| 标段编辑（子级） | `/jyzxt/jsgcjy/gkzb/jysq/jsgczbxmbdbjcly/cu/<标段id>?zbxmlx=..&zblx=..&dbms=..` |
| 一对多查询端点 | `/jyzxt/jsgcjy/gkzb/jysq/jsgczbxmbdbjcly/query12m/<name>` |

> 仓库里**没有** `jsgczbxmsqcxcly` 这个路径，那是模型名（建设工程招标项目）和
> 页面后缀（cxcly 处理页）拼在一起造成的错觉。别按名字猜路径，grep
> `generated/Abstract*.java` 的 `PATH` 常量。

## 页签

| 页签名 | 锚点 | 面板 id | 备注 |
|---|---|---|---|
| 标段信息 | `#tab179798141` | `panel180941525` | 主字段 + 标段价格等 |
| 资格评审 | `#tab180590082` | `panel158703939` | 一对多内嵌表 |
| 形式评审、响应性评审 | `#tab180611183` | `panel171701846` | 一对多内嵌表 |
| 详细评审 | `#tab180638754` | — | 含评标方法/委员会 + 技术评审 + 商务评审 |
| 定标配置 | `#tab0a15eg2a86c9` | `panel0a15eg2g0hej` | `dbms=03` 时才显示（custom.jsp 里 `$("a[href='#tab0a15eg2a86c9']").show()`） |

## 内嵌列表面板

| 面板 | 端点后缀 | 新增按钮 id | 行内控件名 | 真实表 |
|---|---|---|---|---|
| `panel158703939` 投标人资格评审项目 | `tbrzgtj` | `#action166334845` | `IE-tbrzgtj-<row>-{xh,xmmc,nryq}` | `jsgczbtbrzgtj` |
| `panel171701846` 形式评审、响应性评审项目 | `fhxsc` | `#action176055052` | `IE-fhxsc-<row>-{xh,xmmc,nryq}` | `jsgczbtbrfhxsc` |
| `panel738925012` 技术评审项目 | `jsyqb` | `#action743941718` | `IE-jsyqb-<row>-{xh,psxmmc,pssm}` | `jsgczbxmjsyq` |
| `panel555117720` 商务评审项目 | `swyqb` | `#action556142065` | `IE-swyqb-<row>-{xh,psxmmc,pssm}` | `jsgczbxmswyq` |
| `panel965844230` 招标文件 | `zbwj` | —（走上传按钮） | — | `jsgczbxmbdfj`（`category='zbwj'`） |
| `panel09ecdg7hjee2` 投标总价量化表 | `zjlhb` | `#action09ecdg8972hb` | `IE-zjlhb-<row>-{xh,lhys,lhbz,zgtjw}` | **`jsgczbxmtbzjlhb`** |

行内控件的 `<row>` 是全局递增计数（`$.listPanelImplementor.loopRowNum`），**不是 1/2/3**。
2026-11-10 实测：同一页依次新增 资格评审→形式评审→量化表，三张表拿到的是
`tbrzgtj-3` / `fhxsc-4` / `zjlhb-5` —— 计数在**整页**范围内递增，不按表各自从 1 开始。

另有同页但本次未录的表：`fhxscdexf`（形式评审第二信封）、`jspsb`/`swpsb`（技术/商务评分表，
`initHidden()` 里被隐藏）、`qtyspfb`、`dbjpz`（多报价配置）。
**这些表的必填性随评标办法变化**：06 定性评审不需要 zjlhb/qtyspfb/dbjpz；
**04 经评审的最低投标价法反过来必须录入 ≥1 行 zjlhb**（`JsgcbdbjService` L937-940）。

## 主表字段 name（标段实体 `jsgczbxmbd`）

| 中文 | name | 列 |
|---|---|---|
| 标段分类 | `bdflCode` | A08=工程施工 |
| 标段名称 | `bdmc` | |
| 标段地址 | `bddz` | |
| 招标内容（模型 nameCN 是"建设内容和规模"） | `jsnrhgm` | 长文本，≥50 汉字 |
| 招标范围及规模 | `zbfw` | 长文本，≥50 汉字 |
| 标准工期(日历天) | `bzgqrlt` | |
| 保证金(元) | `bzjy` | |
| 履约保证金(%) | `lybzj` | 可空 |
| 合同估算价(元) | `htgsjy` | |
| 招标控制价(元) | `zbkzjy` | |
| **是否暗标** | `sfab` | radio `value="true"/"false"` |
| 资格审查方式 | `zgscfsCode` | 02=资格后审 |
| 投标人类型 | `sgrlxCode` | 2=企业 |
| 报价类型 | `bjlxCode` | 01=投标报价 |
| 标书副本数量 | `bsfbsl` | |
| 评标委员会人数 | `pbwyhrs` | |
| 定标办法 | `dbbfCode` | 01=单轮票决法 |
| 定标委员会人数 | `dbwyhrs` | |
| 是否暂存 | `sfzc` | `保存`动作置 false |

子表实体：`pfff`（评标方法配置，`pbbfCode` 评标办法）、`zwxx`（招文信息）。

## 动作按钮

- 暂存 / 保存 / 关闭在页面底部，id 形如 `#action09bje8ifhkgf` / `#action0a0259ef61c3`。
  **id 要在目标 JSP 里 grep**，本页实测保存按钮 id 曾为 `#action0a0259ef61c3`。
- 点「保存」弹确认框（「是否保存项目标段？」），肯定按钮 selector = `button.confirm`。

## custom.jsp 里的联动（会影响录数）

`jsgczbxmbdbjcly.custom.jsp`：

- `initHidden()`：按评标办法隐藏技术/商务评分表、总价量化表、多报价配置等面板；
  **技术评审面板 `#panel738925012`、商务评审面板 `#panel555117720` 默认也是隐藏的**，
  选定评标办法（06/07/08 分支）后才 `show()`。看不到面板先检查评标办法选没选。
- `changePfbf_A01_A02()`：评标办法 change 时重建明细面板——**这就是 `select_option`
  对 `pfff.pbbfCode` 特别慢的原因**（会做大量 DOM 操作）。
- `dbms=="03"`（评定分离）：隐藏评标办法 01/02/03/04/05，只留 06/07/08；
  A03/A06 只留 06，其余留 06/07/08。同时显示「定标配置」页签。
- `initData(zbxmlx, zblx)`：本项目里**已被注释掉**（`custom.jsp:349-351`），
  所以新建标段时资格条件/符合性审查**不会自动预填行**，必须手工点「新增」。
  换项目要先确认这段是不是还活着。
- 父级「交易申请」页 `jsgcjysqcly.custom.jsp:350-357` 用 `z.openWindow` 打开标段编辑页，
  URL 带 `?zbxmlx=&zblx=&dbms=`。**从父级点进去**，别手工拼 URL。
  ⚠️ 但父级**列表行内的「编辑」链接**（`#action454240462`）URL 是 `cu/{{entity.id}}?`，
  **不带这三个参数** → 服务端跳过分支校验 → 假落库。
  要么用 custom.jsp 的 `openBdbj` 入口，要么手动补成
  `cu/<id>?zbxmlx=A01&zblx=07&dbms=03`。

## 浏览器操作要点（标段页专用）

通用版见 `spl5-browser-test` skill 的 `browser-tool-tactics.md`；这里只记标段页特有的。

### 金额/数字字段：`.numberFormat()` 会吃掉 `fill`

`jsgczbxmbdbjcly.jsp` 对 `bzjy` / `zbkzjy` / `htgsjy` / `bsfbsl` 等调了
`$("#fieldXXX").numberFormat()`。`jquery.z.adhoc.js` 的 `numberFormat`：

- `focus → $this.select()`，且用 `.val()`（**property**）写值、`blur → formatInput` 重写。
- **写**：`fill` / `type` 都会把新值**追加**到旧值后面（填 `1000000` → `10000001000000`）。
  正确姿势：`click` 聚焦 → `press_key{key:"a",modifiers:["Control"]}` 全选 →
  `press_key{key:"Backspace"}` 清空 → `type` 新值 → 点别处 blur 触发格式化。
  ⚠️ `press_key{key:"Control+a"}` **无效**，必须写成 `key:"a", modifiers:["Control"]`。
- **读**：**只有 `screenshot` 可靠**。
  - `query(kind="dom")` 返回 outerHTML，里面的 `value` 是 HTML 属性，
    **不代表实际值**（插件写的是 property）→ 读出来常常是 `""`。
  - `query(kind="editable")` 的 `attributes.value` **同样不可靠**（2026-11-10 实测：
    `pfff.jsf` 页面预填 `0`，DOM 与 editable 都读出空/非 live 值）。
  - `query(kind="dom")` 对 **textarea 读不到内容**（内容是子节点，工具摘不出来）；
    `kind="text"` 对 readonly textarea 也返回空 → 长文本只能靠截图或查库核对。

### 长文本弹窗的「确定」在**另一个 frame**里（关键）

标段页的长文本弹窗（`longTextInputDialog`）把「确定/取消」放在
`.ui-dialog-footer` 里，但 **`.ui-dialog` 这个容器选择器匹配不到它**：

- `query(kind="dom", selector=".ui-dialog button")` → 只返回 `button.ui-dialog-close`（×关闭）。
- `query(kind="dom", selector="button.ui-dialog-autofocus")` → 能返回
  `<button type="button" i-id="ok" autofocus="" class="ui-dialog-autofocus">确定</button>`，
  但 **`click` 它不生效**（点了没反应，弹窗还在）。
- **可靠做法**：`query(kind="semantic", text="确定")` 拿到带 `frame` 的 opaque ref，
  再 `click(ref)` —— 这样点的是那个 frame 里**真正可见**的按钮。
  2026-11-10 实测：语义查询返回的 ref 直接 click，一次就关掉了弹窗。

同理，标段页上传弹窗里的 `input[type=file]` 也在 **iframe** 内，
主文档 `query(kind="dom", selector="input[type=file]")` 会报 "Element not found"，
必须 `inspect` 找到那个 iframe 里的 `input` ref，再用 `upload_files(ref, paths)`。

### 底部动作按钮（暂存/保存/关闭）没有稳定 id

这三个按钮是裸 `<button>`（无 `id`/`name`），且 `query(kind="semantic", text="保存")`
会**误匹配到表单字段**（页面上有 label 含"保存"二字）。可靠做法：

1. 先 `screenshot` 看清底部按钮条的**像素位置**；
2. `click(input={position:{x,y}})` 按坐标点。

确认框的「是」用 `button.confirm` 可以直接点（class="confirm btn btn-primary"）；
长文本弹窗的「确定」用 `button.ui-dialog-autofocus` **不行**（见上）。

### 暂存 / 保存 的差异（2026-11-10 实测）

| 动作 | 确认框 | 落库结果 |
|---|---|---|
| 暂存 | 「是否保存项目标段？」→ 点「是」 | 主表落库，`sfzc=1`；**子表不落库** |
| 保存 | 同上 | 主表落库，`sfzc=0`；子表（资格/形式/量化表）**一并落库** |

⚠️ **暂存后刷新，子表全空、评标办法也被重置为 `--请选择--`**（这是正常的：
暂存只提交主表）。要验证子表必须走「保存」。

### 服务端校验的报错横幅

点「保存」后若校验不过，页面顶部会出现**橙色横幅**列出全部错误项
（如「进入评审数量必须大于等于3；请填写抽取下浮系数低值；请填写抽取下浮系数高值；请正确填写号球数」）。
横幅**不会自动消失**，改完字段后仍会留着上一次的内容 —— 以横幅文字是否变化判断本次结果，
或直接查库确认。

### 标段页字段 id 对照（汕尾）

| 字段 | name | input id |
|---|---|---|
| 标段分类 | `bdflCode` | 9 个 checkbox（label=规划/投资策划与决策/勘察/设计/监理/工程造价/项目管理(含代建)/**工程施工**/其他工程） |
| 标段名称 | `bdmc` | `field181502127` |
| 标段地址 | `bddz` | `field181502299` |
| 招标内容 | `jsnrhgm` | `field815023510`（textarea） |
| 招标范围及规模 | `zbfw` | `field997960040`（textarea） |
| 标准工期 | `bzgqrlt` | `field997960251` |
| 招标控制价 | `zbkzjy` | `field846679418` |
| 合同估算价 | `htgsjy` | `field0a74982ih0b1` |
| 保证金 | `bzjy` | `field846677615` |
| 标书副本数量 | `bsfbsl` | `field251740996` |
| 是否暗标 | `sfab` | radio `value=true/false` |

> 这些 id 随模型重新生成会变。**优先用 `name` 定位**（`#field846677615` 之类的 id 只是备用），
> 换项目时重新 grep 生成的 JSP。
