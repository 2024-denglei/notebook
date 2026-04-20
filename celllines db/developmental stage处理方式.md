我修改了 

cellosaurus_full_extract.py 中 `developmental stage` 字段的值，使其全部使用 **EFO 本体中 `efo:EFO_0000399` (developmental stage) 分支下真实存在的术语**，而不是之前代码中自造的术语。

### 问题：之前使用了 EFO 中不存在的术语

通过搜索 

ontologies/efo.obo，我确认以下术语 **不存在** 于 EFO 本体中：

|之前使用的（错误）|EFO 中实际的术语|本体 ID|
|---|---|---|
|`infant stage`|**`infant`**|EFO:0001355|
|`child stage`|**`juvenile stage`**|UBERON:0034919|
|`late juvenile stage`|**`juvenile stage`**|UBERON:0034919|
|`fetal stage`|**`late embryonic stage`**|UBERON:0007220|
|`embryonic stage`|**`embryo stage`**|EFO:0007725|
|`neonatal stage`|**`neonate`**|EFO:0001372|
|`blastocyst stage`|**`blastula stage`**|UBERON:0000108|
|`morula stage`|**`cleavage stage`**|UBERON:0000107|

### 修改 1：estimate_developmental_stage() 函数

这个函数根据数值型年龄字符串推断发育阶段。

**改进点：**

- 增加 `FW`/`FD`/`FM`（胎儿周/天/月）识别 → 返回 `late embryonic stage`
    
- 用 `re.search(r'(\d+)\s*Y', s)` 提取年龄，可以处理 `>65Y`、`<10Y` 等格式
    
- 增加 `M`/`W`/`D` 单位处理（如 `<1D` → `neonate`，`<1M` → `neonate`）
    
- 年龄分段：`<3Y` → `infant`，`3-17Y` → `juvenile stage`，`≥18Y` → `adult`
    

### 修改 2：`_CELLOSAURUS_DEV_STAGE_MAP` 映射表

将 Cellosaurus AG 字段的文本值映射到正确的 EFO 术语。

**新增条目：** `children`、`prepubertal`、`immature`、`sub-adult`、`subadult`、`young`、`mature`、`adult stage`、`blastula stage`、`gastrula stage`、`late embryonic stage`

### 验证结果

重新生成后，用 Python 统计了 `developmental stage` 列的值分布：

adult: 69,041               (EFO:0001272)

not available: 27,373        (保留字)

juvenile stage: 10,853       (UBERON:0034919)

late embryonic stage: 5,306  (UBERON:0007220)

blastula stage: 4,875        (UBERON:0000108)

infant: 3,572                (EFO:0001355)

neonate: 1,368               (EFO:0001372)

cleavage stage: 14           (UBERON:0000107)

所有 122,402 条记录的 `developmental stage` 值现在都是 EFO 本体中真实存在的术语。