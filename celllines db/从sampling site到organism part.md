
### 原始数据长什么样

Cellosaurus 里每个 cell line 有一行这样的信息：

CC   Derived from site: In situ; Peripheral blood; UBERON=UBERON_0013756

这里有三样东西：

- **类型**：`In situ`（原位）或 `Metastatic`（转移）
    
- **采样部位名称**：`Peripheral blood`（外周血）
    
- **UBERON 编号**：`UBERON_0013756`
    

**Sampling site** 直接取名称部分，就是 `Peripheral blood`。

但 **Organism part** 不能直接用这个名称——"外周血"太具体了，我们想要的是更概括的说法，比如 `Blood`（血液）。

---

### 怎么从 "Peripheral blood" 得到 "Blood"？

UBERON 本体就像一棵**家族树**，每个解剖学术语都有"父辈"：

Peripheral blood

    ↑ part_of（属于）

Blood

    ↑ is_a（是一种）

Bodily fluid（体液）

我的做法分两步：

**第一步：判断这个术语"是什么级别"**

用 `is_a`（"是一种"）关系往上查，看它属于哪个大类：

|大类|例子|
|---|---|
|**器官（organ）**|肺、脑、肝、肾|
|**组织/体液（tissue/fluid）**|骨髓、血液|
|**器官组成部分（organ part）**|宫颈、子宫内膜、结肠|

**第二步：决定要不要"往上爬"**

- 如果术语**本身**就是器官/组织/体液/器官组成部分 → **直接用它**，不用爬
    
- 如果不是 → 沿着 `part_of`（"属于"）关系**往上爬**，找到第一个属于以上大类的祖先
    

---

### 举几个实际例子

|Sampling site|UBERON 编号|推导过程|Organism part|
|---|---|---|---|
|Lung|`0002048`|lung is_a→ organ ✅ 自己就是器官|**Lung**|
|Peripheral blood|`0013756`|不是器官/组织/organ part → part_of→ blood → blood is_a→ bodily fluid ✅|**Blood**|
|Bone marrow|`0002371`|bone marrow is_a→ tissue ✅ 自己就是组织|**Bone Marrow**|
|Uterus, cervix|`0000002`|cervix is_a→ neck of organ → ... → organ part ✅ 自己就是器官组成|**Uterine Cervix**|
|Fetal kidney|`0002113`|kidney is_a→ organ ✅|**Kidney**|
|Islet of Langerhans|`0000006`|不是器官/组织/organ part → part_of→ pancreas → pancreas is_a→ organ ✅|**Pancreas**|

**一句话概括**：先看这个部位本身够不够"大"（是器官/组织/体液/器官组成部分），够大就直接用；不够大就沿着"属于"关系往上爬，找到第一个够大的祖先。

Feedback submitted