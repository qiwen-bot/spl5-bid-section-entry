# 标段落库核对（汕尾建设工程）

## 一条 SQL 看完七张表

```sql
SELECT
  (SELECT COUNT(*) FROM jsgczbxmbd       WHERE id='<标段id>' AND deleted=0) bdb,
  (SELECT COUNT(*) FROM jsgczbtbrzgtj    WHERE ssxmbdId='<标段id>') tbrzgtj,
  (SELECT COUNT(*) FROM jsgczbtbrfhxsc   WHERE ssxmbdId='<标段id>') fhxsc,
  (SELECT COUNT(*) FROM jsgczbxmjsyq     WHERE ssxmbdId='<标段id>') jsyqb,
  (SELECT COUNT(*) FROM jsgczbxmswyq     WHERE ssxmbdId='<标段id>') swyqb,
  (SELECT COUNT(*) FROM jsgczbxmtbzjlhb  WHERE ssxmbdId='<标段id>') zjlhb,
  (SELECT COUNT(*) FROM jsgczbxmbdfj     WHERE ownerId='<标段id>' AND category='zbwj') zbwj
```

期望值：`bdb=1`、其余全为 `1`。

> **表名最容易看串的三组**（2026-11-10 实测各踩一次，查错表会得出「没落库」的假结论）：
>
> | 面板 | 真实表 | 关键列 |
> |---|---|---|
> | 技术评审项目 / 商务评审项目 | `jsgczbxmjsyq` / `jsgczbxmswyq` | `ssxmbdId`, `psxmmc`, `pssm` |
> | **投标总价量化表** | **`jsgczbxmtbzjlhb`** | `ssxmbdId`, `lhys`, `lhbz`, `zgtjw` |
> | 评标办法配置 | `jsgczbxmpfff` | `id`（由 `jsgczbxmbd.pfffId` 关联） |
>
> - `jsgczbxmtbjsyqb` / `jsgczbxmtbswyqb` 是另一组实体（技术要求表/商务要求表），查它们会得到 0 行。
> - `jsgczbxmbazl` **不是**投标总价量化表（查它永远 0 行；该表在本库为空）。
> - 评标办法不在主表上，要 `JOIN jsgczbxmpfff p ON p.id=b.pfffId`；`jsgczbxmpfff` 没有 `ssxmbdId` 列。
> - `jsgczbxmbdfj` 没有 `deleted` 列（用 `category` 过滤；`zbwj`=招标文件）。

## 逐字段核对标段主表

```sql
SELECT CONCAT('bdflCode=',IFNULL(bdflCode,'NULL')) a,
       CONCAT('jsnrhgm.len=',IFNULL(CHAR_LENGTH(jsnrhgm),-1)) b,
       CONCAT('zbfw.len=',IFNULL(CHAR_LENGTH(zbfw),-1)) c,
       CONCAT('bzgqrlt=',IFNULL(bzgqrlt,'NULL')) d,
       CONCAT('bzjy=',IFNULL(bzjy,'NULL')) e,
       CONCAT('htgsjy=',IFNULL(htgsjy,'NULL')) f,
       CONCAT('zbkzjy=',IFNULL(zbkzjy,'NULL')) g,
       CONCAT('sfab=',CAST(sfab AS UNSIGNED)) h,
       CONCAT('zgscfsCode=',IFNULL(zgscfsCode,'NULL')) i,
       CONCAT('sgrlxCode=',IFNULL(sgrlxCode,'NULL')) j,
       CONCAT('bjlxCode=',IFNULL(bjlxCode,'NULL')) k,
       CONCAT('pbwyhrs=',IFNULL(pbwyhrs,'NULL')) l,
       CONCAT('dbwyhrs=',IFNULL(dbwyhrs,'NULL')) m,
       CONCAT('dbbfCode=',IFNULL(dbbfCode,'NULL')) n,
       CONCAT('bsfbsl=',IFNULL(bsfbsl,'NULL')) o,
       CONCAT('sfzc=',CAST(sfzc AS UNSIGNED)) p,
       CONCAT('deleted=',CAST(deleted AS UNSIGNED)) q
FROM jsgczbxmbd WHERE id='<标段id>'
```

> **`sfab` / `sfzc` / `deleted` 是 `bit(1)`，必须 `CAST(... AS UNSIGNED)`。**
> 直接 `SELECT sfab` 能拿到 "1"/"0"，但一旦套进 `CONCAT`/`IFNULL`，
> MySQL 会把它当二进制串，控制字节打印出来是**空白**——看起来像"没值"，
> 实际是 1。这是判"真保存"最关键的三个信号，绝不能用 CONCAT 形式读。
> （`dbq.ps1` 已加兜底：遇到不可见控制字符会打成 `0x..` 十六进制，但 SQL 里还是显式 CAST 更清楚。）

注意列名容易看串：`jsnrhgm` 是**建设内容和规模**，`zbfw` 是**招标范围**，
`bzgqrlt` 是**标准工期**，`bzjy` 是**保证金**，`htgsjy` 是**合同估算价**，`zbkzjy` 是**招标控制价**。

## 评标方法配置

```sql
SELECT p.pbbfCode, p.jglxCode, p.jsf, p.swf, p.jgf,
       CAST(p.sfxzpssl AS UNSIGNED) sfxzpssl, p.jrpssl, p.hqs, p.cqxfxsdz, p.cqxfxsgz
FROM jsgczbxmbd b JOIN jsgczbxmpfff p ON p.id=b.pfffId
WHERE b.id='<标段id>'
```

`pbbfCode`：`04`=经评审的最低投标价法、`06`=定性评审、`01`=综合评分法。

**`sfxzpssl`（是否限制评审数量）决定另外三个字段是否必填**（`JsgcbdbjService.java` 04 分支 L925-936）：
`sfxzpssl != false` 时，下浮系数低值/高值/号球数**全部必填**，且「进入评审数量」必须 ≥3（L399）。
房建市政项目（`bdflCode=A01`）+ 04 评标办法时，把「是否限制评审数量」选**否**即可全部跳过。

## 04 分支的必填矩阵（2026-11-10 实测）

`pbbfCode='04'`（经评审的最低投标价法）需要：

| 字段 | 必填条件 | 页面 name |
|---|---|---|
| 报价类型 = 投标报价 | 恒定 | `bjlxCode`（主表） |
| 投标总价量化表 ≥1 行 | 恒定 | `IE-zjlhb-<row>-{xh,lhys,lhbz,zgtjw}` |
| 号球数 / 下浮系数低值 / 高值 | 仅当 `sfxzpssl != false` | `pfff.hqs` / `pfff.cqxfxsdz` / `pfff.cqxfxsgz` |
| 进入评审数量 ≥3 | 仅当 `sfxzpssl == true` | `pfff.jrpssl` |

最省事的做法：把「是否限制评审数量」选 **否**（`#tag1794275687692False`，name=`pfff.sfxzpssl`），
只填评标办法 + 评委会人数 + 1 行投标总价量化表即可通过服务端校验。

## 长文本是否重复

弹窗写入可能是追加。核对每条评审说明的实际长度与内容：

```sql
SELECT xh, xmmc, CHAR_LENGTH(nryq) len, nryq FROM jsgczbtbrzgtj  WHERE ssxmbdId='<标段id>'
UNION ALL
SELECT xh, xmmc, CHAR_LENGTH(nryq), nryq FROM jsgczbtbrfhxsc WHERE ssxmbdId='<标段id>'
UNION ALL
SELECT xh, psxmmc, CHAR_LENGTH(pssm), pssm FROM jsgczbxmjsyq  WHERE ssxmbdId='<标段id>'
UNION ALL
SELECT xh, psxmmc, CHAR_LENGTH(pssm), pssm FROM jsgczbxmswyq  WHERE ssxmbdId='<标段id>'
```

`len` 明显接近单份文本长度的两倍 → 内容重复了，重录一次。

## 怎么执行

连接信息取自 `swggzyjy2022/src/main/resources/application.properties` 的 `spring.datasource.*`。
执行方式与通用流程见 **`spl5-browser-test` skill** 的核对参考文档与 `scripts/dbq.ps1`
（PowerShell + JDK 21，从 `target\*.jar` 里取 mysql 驱动，无需安装 mysql client）。

## 先确认线上真实表结构

仓库里的 `init.sql` 可能早于线上部署版本（缺后加的列、缺新表）。查表前先跑：

```sql
SELECT table_name, GROUP_CONCAT(column_name ORDER BY ordinal_position) cols
FROM information_schema.columns
WHERE table_schema=DATABASE() AND table_name LIKE 'jsgczbxm%'
GROUP BY table_name
```

汕尾测试环境实测：`jsgczbxmbd` 有 `sfab`/`sfzc`/`dbbfCode`/`dbwyhrs`/`zbkzjy`/`htgsjy`/`bzjy`，
`jsgczbxmswyq` 表存在——但这些**不保证**在仓库的 init.sql 里也有，别照抄那份 DDL。

## 列名别看串

| 列 | 含义 |
|---|---|
| `jsnrhgm` | 招标内容（模型 nameCN 写作"建设内容和规模"） |
| `zbfw` | 招标范围及规模 |
| `bzgqrlt` | 标准工期(日历天) |
| `bzjy` | 保证金(元) |
| `htgsjy` | 合同估算价(元) |
| `zbkzjy` | 招标控制价(元) |
| `sfab` | 是否暗标（1=是） |
| `sfzc` | 是否暂存（保存动作置 0，暂存置 1） |
| `ssxmbdId` | 子表 → 标段 id |
| `sszbxmId` | 标段 → 交易申请 id |
