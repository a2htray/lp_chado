### 介绍

`semantic networks and ontologies, depending on which terminology you prefer`

CV，Controlled Vocabularies，根据不同的术语结构，定义通用的分类结构，用于定义语言化的分类结构。

CV，直译"控制词汇"

`The schema reflects the data model of OBO and of the OBO Edit (http://oboedit.org/) tool currently used by these projects`

OBO 文件表示关系，存储在数据库中的结构，即为 CV 模块所定义的表结构。

<!--TODO There is a bridge layer in the directory modules/cv/bridges/ 看源码-->

#### cvterm

`cvterms are related to one another via cvterm_relationship`

一个特性进行分类定义，分类与分类间存在某种关系。而定义的术语存储在 cvterm 表，cvterm 表中的记录通过 cvterm_relationship 表进行关联，用于表示两者之间的关系。可以用利用 `图` 或者 `语义化网络结构` 进行描述。两者关系也定义成一个 cvterm，并存储在 cvterm 表。

#### cvtermpath

`For a typical database, which may only have relations isa, part_of and develops_from, we will end up with 3 sets of paths`

该表中数据是由 cvterm 和 cvterm_relationship 表中的关系计算并填充而来，只计算 `is_a` 、`part_of` 和 `develops_from` 三种关系。









