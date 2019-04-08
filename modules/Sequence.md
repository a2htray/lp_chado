### 介绍

Chado 数据库最核心的模块是 Sequence 模块，而该模块中最重要的表为 `feature` 表，用于记录序列特征。Chado 将特征定义为各种信息聚合后一个区域 `region` 的特征表现。术语 `region` 可以是一个分子，也可以是两个或多个的重叠。`region` 可以被预先定义。

### Feature

Chado 并不区分序列(`sequence`)和序列特征(`sequence feature`)。因为在理论上，特征是序列的一个片段，而序列的一个片段也是叫序列。在表 feature 中都以一行的数据。

特征 `feature` 包含多种类，如 `gene, exon, transcript, regulatory region, chromosome, sequence variation, polypeptide, protein domain 和 cross-genome match regions`。

特征类型由序列分类信息 [Sequence Ontology](http://www.sequenceontology.org/)) 决定。