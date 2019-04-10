### Table of Contents

* [Module `General`](#module-general)
    * [Table `tableinfo`](#table-tableinfo)
    * [Table `db`](#table-db)
    * [Table `dbxref`](#table-dbxref)
    * [View `db_dbxref_count`](#view-db_dbxref_count)
    * [Function `store_db`](#function-store_db)
* [Module `CV`](#module-cv)
    * [Table `cv`](#table-cv)
    * [Table `cvterm`](#table-cvterm)
    * [Table `cvterm_relationship`](#table-cvterm_relationship)
    * [Table `cvtermpath`](#table-cvtermpath)
    * [Table `cvtermsynonym`](#table-cvtermsynonym)
    * [Table `cvterm_dbxref`](#table-cvterm_dbxref)
    * [Table `cvtermprop`](#table-cvtermprop)
    * [Table `dbxrefprop`](#table-dbxrefprop)
    * [Table `cvprop`](#table-cvprop)
    * [Table `chadoprop`](#table-chadoprop)
    * [Table `dbprop`](#table-dbprop)
    * [View `cv_root`](#view-cv_root)
    * [View `cv_leaf`](#view-cv_leaf)
    * [View `common_ancestor_cvterm`](#view-common_ancestor_cvterm)
    * [View `common_descendant_cvterm`](#view-common_descendant_cvterm)
    * [View `stats_paths_to_root`](#view-stats_paths_to_root)
    * [View `cv_cvterm_count`](#view-cv_cvterm_count)
    * [View `cv_cvterm_count_with_obs`](#view-cv_cvterm_count_with_obs)
    * [View `cv_link_count`](#view-cv_link_count)
    * [View `cv_path_count`](#view-cv_path_count)
* [Module `Contact`](#module-contact)
    * [Table `contact`](#table-contact)
    * [Table `contactprop`](#table-contactprop)
    * [Table `contact_relationship`](#table-contact_relationship)
* [Module `pub`](#module-pub)
    * [Table `pub`](#table-pub)
    * [Table `pub_dbxref`](#table-pub_dbxref)
    * [Table `pubauthor`](#table-pubauthor)
    * [Table `pubprop`](#table-pubprop)
    * [Table `pubauthor_contact`](#table-pubauthor_contact)

### Module `General`

#### Table `tableinfo`

Column | Type | Modifiers | Desc |
---|---|---|---|
tableinfo_id | bigserial | primary key not null | 自增主键 |
name | varchar(30) | not null unique| 表名 |
primary_key_column | varchar(30) | null | 当前所记录表的主键字段 |
is_view | int | not null default 0 | 是否是视图 |
view_on_table_id | bigint | null | - |
superclass_table_id | bigint | null | 父表ID |
is_updateable | int | not null default 1 | 是否可更新 |
modification_date | date | not null default now() | 更新时间 |

* primary key (tableinfo_id)
* constraint tableinfo_c1 unique (name)

#### Table `db`

生物信息学数据库信息，典型的如: FlyBase, GO, UniProt, NCBI, MGI。

Column | Type | Modifiers | Desc |
---|---|---|---|
db_id | bigserial | not null primary key | 自增主键 |
name | varchar(255) | not null unique | 数据库名 |
description | varchar(255) | null | 描述 |
urlprefix | varchar(255) | null | 前缀信息 |
url | varchar(255) | null | URL 信息 |

* primary key (db_id)
* constraint db_c1 unique (name)

#### Table `dbxref`

唯一的、全局的、公开的指示器表，用于指向数据库特定的数据。

实例形如 `<DB>:<ACCESSION>` 或 `<DB>:<ACCESSION>:<VERSION>`

Column | Type | Modifiers | Desc |
---|---|---|---|
dbxref_id | bigserial | not null primary key | 自增主键 |
db_id | bigint | not null / index | 外键，表`db`中的`db_id` |
accession | varchar(1024) | not null / index | 指示器部分信息，用于拼接成完整的指示器 |
version | varchar(255) | not null default '' / index | 版本信息 |
description | text | - | 描述信息 |

* primary key (dbxref_id)
* foreign key (db_id) references db (db_id) on delete cascade INITIALLY DEFERRED
* index dbxref_idx1 on dbxref (db_id)
* index dbxref_idx2 on dbxref (accession)
* index dbxref_idx3 on dbxref (version)

#### View `db_dbxref_count`

视图，每一个 db 所包含的 dbxref 的个数。

Column | Type | Modifiers | Desc |
---|---|---|---|
name | varchar(255) | - | db 名称 |
num_dbxrefs | integer | - | 包含指示器的个数 |

#### Function `store_db`

用于`db`表新增记录，不存在则新增，返回 `db_id`。

```sql
-- @param VARCHAR db的名称
-- @return INTEGER 记录ID
SELECT * FROM store_db('Go');
```

#### Function `store_dbxref`

`dbxref`表新增记录，不存在则新增，内部先调用函数`store_db`向`db`表插入记录，返回 `dbxref_id`。

```sql
-- @param VARCHAR db名称
-- @param VARCHAR accession值
-- @return INTEGER 记录ID
SELECT * FROM store_dbxref('Go', 'Go accession');
```

### Module `CV`

对不同领域的术语进行语义分类，使术语的定义更为通用

#### Table `cv`

分类条目信息表，由 `OBO 文件`导入

A controlled vocabulary or ontology. A cv is composed of cvterms (AKA terms, classes, types, universals - relations and properties are also stored in cvterm) and the relationships between them.

Column | Type | Modifiers | Desc |
---|---|---|---|
cv_id | bigserial | not null | 自增主键 |
name | varchar(255) | not null unique | 名称 |
definition | text | - | 自定信息 |

* primary key (cv_id)
* constraint cv_c1 unique (name)

#### Table `cvterm`

存放术语信息

分类术语表，用于表示关系和属性，与表 `cvterm_relationships` 相互作用

与表`cvterm_relationships`可以画有向无环图DAG，构建语义化网络图

表`cvterm`的记录充当于DAG的顶点，表`cvterm_relationship`的记录充当DAG的边

The collection of cvterms and cvterm_relationships can be considered to constitute vertices and edges in a graph. 

A term, class, universal or type within an ontology or controlled vocabulary.  This table is also used for relations and properties. cvterms constitute nodes in the graph defined by the collection of cvterms and cvterm_relationships.

Column | Type | Modifiers | Desc |
---|---|---|---|
cvterm_id | bigserial | not null primary key | 自增主键 |
cv_id | bigint | not null / index / foreign key | 表`cv`的`cv_id`，属于哪一个 cv |
name | varchar(1024) | not null / index | 名称 |
definition | text | - | 定义信息 |
dbxref_id | bigint | not null / foreign key / unique | 表`dbxref`的`dbxref_id`，主指示器引用 |
is_obsolete | int | not null default 0 | 如果为1，则该 term 的主 dbxref与其它不同 |
is_relationshiptype | int | not null default 0 | indicate whether this cvterm is an actual term/class/universal or a relation |

* primary key (cvterm_id)
* foreign key (cv_id) references cv (cv_id) on delete cascade INITIALLY DEFERRED
* constraint cvterm_c1 unique (name,cv_id,is_obsolete)
* constraint cvterm_c2 unique (dbxref_id)
* create index cvterm_idx1 on cvterm (cv_id);
* create index cvterm_idx2 on cvterm (name);
* create index cvterm_idx3 on cvterm (dbxref_id);

#### Table `cvterm_relationship`

用于链接两个 cvterm，表示两之间的关系

为表`cvterm`的记录进行定义关系的作用，

A relationship linking two cvterms. Each cvterm_relationship constitutes an edge in the graph defined by the collection of cvterms and cvterm_relationships. The meaning of the cvterm_relationship depends on the definition of the cvterm R refered to by type_id. However, in general the definitions are such that the statement "all SUBJs REL some OBJ" is true. The cvterm_relationship statement is about the subject, not the object. For example "insect wing part_of thorax".

subject_id 表示主语信息，type_id 表示谓语信息，object_id 表示宾语信息，即 `什么是什么`，主要用于强调主语的关系

Column | Type | Modifiers | Desc |
---|---|---|---|
cvterm_relationship_id | bigserial | not null primary key | 自增主键 |
type_id | bigint | not null / foreign key / index | `cvterm_id` |
subject_id | bigint | not null / foreign key / index | `cvterm_id` |
object_id | bigint | not null / foreign key / index | `cvterm_id` |

<!-- TODO 理解 type_id subject_id object_id -->

* primary key (cvterm_relationship_id)
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED
* foreign key (subject_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED
* foreign key (object_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED
* constraint cvterm_relationship_c1 unique (subject_id,object_id,type_id)
* create index cvterm_relationship_idx1 on cvterm_relationship (type_id);
* create index cvterm_relationship_idx2 on cvterm_relationship (subject_id);
* create index cvterm_relationship_idx3 on cvterm_relationship (object_id);

#### Table `cvtermpath`

用于定义通路闭环的问题

<!-- TODO cvterm 与 cvtermpath 之间的关系 -->

Column | Type | Modifiers | Desc |
---|---|---|---|
cvtermpath_id | bigserial | not null primary key | 自增主键 |
type_id | bigint | foreign key | `cvterm_id` |
subject_id | bigint | not null foreign key | `cvterm_id` |
object_id | bigint | not null foreign key | `cvterm_id` |
cv_id | bigint | not null foreign key | `cv_id` |
pathdistance | int | - | - |


* foreign key (type_id) references cvterm (cvterm_id) on delete set null INITIALLY DEFERRED,
* foreign key (subject_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED,
* foreign key (object_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED,
* foreign key (cv_id) references cv (cv_id) on delete cascade INITIALLY DEFERRED,
* constraint cvtermpath_c1 unique (subject_id,object_id,type_id,pathdistance)
* create index cvtermpath_idx1 on cvtermpath (type_id);
* create index cvtermpath_idx2 on cvtermpath (subject_id);
* create index cvtermpath_idx3 on cvtermpath (object_id);
* create index cvtermpath_idx4 on cvtermpath (cv_id);

### Table `cvtermsynonym`

同义词表，同一个术语可能有多个与之同义的术语，`cvterm.name` 作为优先级第一的术语名称

Column | Type | Modifiers | Desc |
---|---|---|---|
cvtermsynonym_id | bigserial | not null primary key | 自增主键 |
cvterm_id | bigint | not null/foreign key/index | `cvterm.cvterm_id` |
synonym | varchar(1024) | not null | 同义名称 |
type_id | bigint | foreign key | `cvterm.cvterm_id` |

<!-- TODO type_id 有什么作用 -->

* primary key (cvtermsynonym_id),
* foreign key (cvterm_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED,
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade  INITIALLY DEFERRED,
* constraint cvtermsynonym_c1 unique (cvterm_id,synonym)
* create index cvtermsynonym_idx1 on cvtermsynonym (cvterm_id);

#### Table `cvterm_dbxref`

`关联表`

一个可以术语可以有多个 dbxref，用于指向其它数据库

Column | Type | Modifiers | Desc |
---|---|---|---|
cvterm_dbxref_id | bigserial | not null/primary key | 自增主键 |
cvterm_id | bigint | not null/foreign key/index | `cvterm.cvterm_id` |
dbxref_id | bigint | not null/foreign key/index | `dbxref.dbxref_id` |
is_for_definition | int | not null default 0 | - |

<!-- TODO 需要知道 cvterm.definition 与 cvterm_dbxref.is_for_definition 的关系 -->

* primary key (cvterm_dbxref_id),
* foreign key (cvterm_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED,
* foreign key (dbxref_id) references dbxref (dbxref_id) on delete cascade INITIALLY DEFERRED,
* constraint cvterm_dbxref_c1 unique (cvterm_id,dbxref_id)
* create index cvterm_dbxref_idx1 on cvterm_dbxref (cvterm_id);
* create index cvterm_dbxref_idx2 on cvterm_dbxref (dbxref_id);

#### Table `cvtermprop`

为 cvterm 增加额外的属性

Column | Type | Modifiers | Desc |
---|---|---|---|
cvtermprop_id | bigserial | not null/primary key | 自增主键 |
cvterm_id | bigint | not null/foreign key/index | `cvterm.cvterm_id` |
type_id | bigint | not null/foreign key/index | `cvterm.cvterm_id` |
value | text | not null default '' | - |
rank | int | not null default 0 | 各个属性的权重 |

* primary key (cvtermprop_id), 
* foreign key (cvterm_id) references cvterm (cvterm_id) on delete cascade, 
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade, 
* unique(cvterm_id, type_id, value, rank) 
* create index cvtermprop_idx1 on cvtermprop (cvterm_id);
* create index cvtermprop_idx2 on cvtermprop (type_id);

#### Table `dbxrefprop`

关于 dbxref 的元数据信息，也是表示一个 dbxref 的各个属性，与表`cvtermprop` 是同构的

Column | Type | Modifiers | Desc |
---|---|---|---|
dbxrefprop_id | bigserial | not null/primary key | 自增主键 |
dbxref_id | bigint | not null/foreign key/index | `dbxref.dbxref_id` |
type_id | bigint | not null/foreign key/index | `cvterm.cvterm_id` |
value | text | not null default '' | - |
rank | int | not null default 0 | 权重 |

* primary key (dbxrefprop_id),
* foreign key (dbxref_id) references dbxref (dbxref_id) INITIALLY DEFERRED,
* foreign key (type_id) references cvterm (cvterm_id) INITIALLY DEFERRED,
* constraint dbxrefprop_c1 unique (dbxref_id,type_id,rank)
* create index dbxrefprop_idx1 on dbxrefprop (dbxref_id);
* create index dbxrefprop_idx2 on dbxrefprop (type_id);

#### Table `cvprop`

cv 的元信息，与表`cvtermprop`、`dbxrefprop` 同构

Column | Type | Modifiers | Desc |
---|---|---|---|
cvprop_id | bigserial | not null/primary key | 自增主键 |
cv_id | bigint | not null/foreign key | `cv.cv_id` |
type_id | bigint | not null/foreign key | `cvterm.cvterm_id` |
value | text | not null default '' | - |
rank | int | not null default 0 | 权重 |

* primary key (cvprop_id)
* foreign key (cv_id) references cv (cv_id) INITIALLY DEFERRED
* foreign key (type_id) references cvterm (cvterm_id) INITIALLY DEFERRED
* constraint cvprop_c1 unique (cv_id,type_id,rank)

#### Table `chadoprop`

存储 chado 数据库本身的信息

Column | Type | Modifiers | Desc |
---|---|---|---|
chadoprop_id | bigserial | not null/primary key | 自增主键 |
type_id | bigint | not null/foreign key | `cvterm.cvterm_id` |
value | text | not null default '' | - |
rank | int | not null default 0 | 权重 |

* primary key (chadoprop_id)
* foreign key (type_id) references cvterm (cvterm_id) INITIALLY DEFERRED
* constraint chadoprop_c1 unique (type_id,rank)

#### Table `dbprop`

数据库，如 GO等 属性信息

Column | Type | Modifiers | Desc |
---|---|---|---|
dbprop_id | bigserial | not null/primary key | 自增主键 |
db_id | bigint | not null/foreign key/index | `db.db_id` |
type_id | bigint | not null/foreign key/index | `cvterm.cvterm_id` |
value | text | not null default '' | - |
rank | int | not null default 0 | 权重 |

* primary key (dbprop_id)
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED,
* foreign key (db_id) references db (db_id) on delete cascade INITIALLY DEFERRED,
* constraint dbprop_c1 unique (db_id,type_id,rank)
* create index dbprop_idx1 on dbprop (db_id);
* create index dbprop_idx2 on dbprop (type_id);

#### View `cv_root`

处于最顶级的 cvterm，即所有 cvterm 的根节点

Column | Type | Modifiers | Desc |
---|---|---|---|
cv_id | - | - | `cvterm.cv_id` |
root_cvterm_id | - | - | cvterm.root_cvterm_id |

#### View `cv_leaf`

没有子节点的 cvterm

Column | Type | Modifiers | Desc |
---|---|---|---|
cv_id | - | - | `cvterm.cv_id` |
root_cvterm_id | - | - | cvterm.root_cvterm_id |

#### View `common_ancestor_cvterm`

任何两个具有共同祖先节点的 cvterm 信息

Column | Type | Modifiers | Desc |
---|---|---|---|
cvterm1_id | - | - | `cvtermpath.subject_id` |
cvterm2_id | - | - | `cvtermpath.subject_id` |
object_id | - | - | `cvtermpath.object_id` |
pathdistance1 | - | - | `cvtermpath.pathdistance` |
pathdistance2 | - | - | `cvtermpath.pathdistance` |
total_pathdistance | - | - | 两子节点的距离和 |

#### View `common_descendant_cvterm`

包含相同子节点的 cvterm 信息表

Column | Type | Modifiers | Desc |
---|---|---|---|
cvterm1_id | - | - | `cvtermpath.subject_id` |
cvterm2_id | - | - | `cvtermpath.subject_id` |
object_id | - | - | `cvtermpath.object_id` |
pathdistance1 | - | - | `cvtermpath.pathdistance` |
pathdistance2 | - | - | `cvtermpath.pathdistance` |
total_pathdistance | - | - | 两子节点的距离和 |

#### View `stats_paths_to_root`

每一个 cvterm 的统计信息 

Column | Type | Modifiers | Desc |
---|---|---|---|
cvterm_id | - | - | - |
total_paths | - | - | 总的路径数 |
avg_distance | - | - | 平均路径长度 |
min_distance | - | - | 最短的一根路径的长度 |
max_distance | - | - | 最长的一根路径的长度 |

#### View `cv_cvterm_count`

过滤 is_obsolute = 0，具有相同 cvterm 名的个数

Column | Type | Modifiers | Desc |
---|---|---|---|
name | - | - | `cv.name` |
num_terms_excl_obs | - | - | 数量 |

#### View `cv_cvterm_count_with_obs`

具有相同 cvterm 名的个数，不进行过滤

Column | Type | Modifiers | Desc |
---|---|---|---|
name | - | - | `cv.name` |
num_terms_incl_obs | - | - | 数量 |

#### View `cv_link_count`

每一个相关联的 cvterm 的数量信息

Column | Type | Modifiers | Desc |
---|---|---|---|
cv_name | - | - | `cv.name` |
relation_name | - | - | `cvterm.name` |
relation_cv_name | - | - | `cv.name` |
num_links | - | - | 数量 |

#### View `cv_path_count`

对于特定 cv、cvterm 具有路径个数表

Column | Type | Modifiers | Desc |
---|---|---|---|
cv_name | - | - | `cv.name` |
relation_name | - | - | `cvterm.name` |
relation_cv_name | - | - | `cv.name` |
num_paths | - | - | 数量 |

<!-- TODO -->

### Module `Feature`

#### Table `feature_relationship`

### Module `Contact`

#### Table `contact`

人员，组织，机构，团体记录表

Column | Type | Modifiers | Desc |
---|---|---|---|
contact_id | bigserial | not null/primary key | - |
type_id | bigint | null/foreign key | 记录的类型，如一般人，实验室等 |
name | varchar(255) | not null/unique | 名称 |
description | varchar(255) | null | 描述信息 |

* primary key (contact_id)
* foreign key (type_id) references cvterm (cvterm_id)
* constraint contact_c1 unique (name)

#### Table `contactprop`

联系记录的属性信息

Column | Type | Modifiers | Desc |
---|---|---|---|
contactprop_id | bigserial | primary key/not null | - |
contact_id | bigint | NOT NULL/foreign key/index | 指向 contact 记录 |
type_id | bigint | NOT NULL/foreign key/index | 属性的类型，`cvterm.cvterm_id` |
value | text | - | 具体信息 |
rank | integer | DEFAULT 0 NOT NULL | 权重、用于排序 |

* CONSTRAINT contactprop_c1 UNIQUE (contact_id, type_id, rank),    
* FOREIGN KEY (contact_id) REFERENCES contact(contact_id) ON DELETE CASCADE,
* FOREIGN KEY (type_id) REFERENCES cvterm(cvterm_id) ON DELETE CASCADE
* CREATE INDEX contactprop_idx1 ON contactprop USING btree (contact_id);
* CREATE INDEX contactprop_idx2 ON contactprop USING btree (type_id);

#### Table `contact_relationship`

用于描述 contact 对象之间的关系表

Column | Type | Modifiers | Desc |
---|---|---|---|
contact_relationship_id | bigserial | not null/primary key | - |
type_id | bigint | not null/foreign key | 关系的类型，`cvterm.cvterm_id` |
subject_id | bigint | not null/foreign key | `contact.contact_id` |
object_id | bigint | not null/foreign key | `contact.contact_id` |

* primary key (contact_relationship_id)
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED,
* foreign key (subject_id) references contact (contact_id) on delete cascade INITIALLY DEFERRED,
* foreign key (object_id) references contact (contact_id) on delete cascade INITIALLY DEFERRED,
* `constraint contact_relationship_c1 unique (subject_id,object_id,type_id)`
* create index contact_relationship_idx1 on contact_relationship (type_id);
* create index contact_relationship_idx2 on contact_relationship (subject_id);
* create index contact_relationship_idx3 on contact_relationship (object_id);

### Module `pub`

有记载的出处，如文章、文档、个人的通信信息

#### Table `pub`

Column | Type | Modifiers | Desc |
---|---|---|---|
pub_id | bigserial | not null/primary key | - |
title | text | - | 描述性的标题 |
volumetitle | text | - | 章节标题 |
volume | varchar(255) | - | 章节 |
series_name | varchar(255) | - | 系列名 |
issue | varchar(255) | - | 解决的问题 |
pyear | varchar(255) | - | 发表的年份 |
pages | varchar(255) | - | 页码信息 |
miniref | varchar(255) | - | - |
uniquename | text | not null/unique - | - |
type_id | bigint | not null/foreign key/index | 出版物的类型，`cvterm.cvterm_id` |
is_obsolete | boolean | default 'false' | - |
publisher | varchar(255) | - | 发表人 |
pubplace | varchar(255) | - | 发表的地方 |

* primary key (pub_id)
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED,
* constraint pub_c1 unique (uniquename)
* CREATE INDEX pub_idx1 ON pub (type_id)

#### Table `pub_relationship`

出版物之间的关系

Column | Type | Modifiers | Desc |
---|---|---|---|
pub_relationship_id | bigserial not null/primary key 
subject_id | bigint | not null/foreign key | `pub.pub_id` |
object_id | bigint | not null/foreign key | `pub.pub_id` |
type_id | bigint | not null/foreign key | `pub.pub_id` |

* primary key (pub_relationship_id)
* foreign key (subject_id) references pub (pub_id) on delete cascade INITIALLY DEFERRED
* foreign key (object_id) references pub (pub_id) on delete cascade INITIALLY DEFERRED
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED
* constraint pub_relationship_c1 unique (subject_id,object_id,type_id)
* create index pub_relationship_idx1 on pub_relationship (subject_id);
* create index pub_relationship_idx2 on pub_relationship (object_id);
* create index pub_relationship_idx3 on pub_relationship (type_id);

#### Table `pub_dbxref`

出版物的数据库信息，即文章的出外

Column | Type | Modifiers | Desc |
---|---|---|---|
pub_dbxref_id | bigserial | not null/primary | - |
pub_id | bigint | not null/foreign key/index | `pub.pub_id` |
dbxref_id | bigint | not null/foreign key/index | `dbxref.dbxref_id` |
is_current | boolean | not null default 'true' | - |

* primary key (pub_dbxref_id),
* foreign key (pub_id) references pub (pub_id) on delete cascade INITIALLY DEFERRED,
* foreign key (dbxref_id) references dbxref (dbxref_id) on delete cascade INITIALLY DEFERRED,
* `constraint pub_dbxref_c1 unique (pub_id,dbxref_id)`
* create index pub_dbxref_idx1 on pub_dbxref (pub_id);
* create index pub_dbxref_idx2 on pub_dbxref (dbxref_id);

#### Table `pubauthor`

Column | Type | Modifiers | Desc |
---|---|---|---|
pubauthor_id | bigserial | not null/primary key | - |
pub_id | bigint | not null/foreign key | `pub.pub_id` |
rank | int | not null | - | 权重 |
editor | boolean | default 'false' | - | 是否是编辑 |
surname | varchar(100) | not null | - | 姓 |
givennames | varchar(100) | - | First name |
suffix | varchar(100) | - | 前缀信息，如 Jr. Sr |

* primary key (pubauthor_id),
* foreign key (pub_id) references pub (pub_id) on delete cascade INITIALLY DEFERRED,
* `constraint pubauthor_c1 unique (pub_id, rank)`

#### Table `pubprop`

出版物的属性记录

Column | Type | Modifiers | Desc |
---|---|---|---|
pubprop_id | bigserial | primary key/not null | - |
pub_id | bigint | NOT NULL/foreign key/index | 指向 pub 记录，`pub.pub_id` |
type_id | bigint | NOT NULL/foreign key/index | 属性的类型，`cvterm.cvterm_id` |
value | text | - | 具体信息 |
rank | integer | DEFAULT 0 NOT NULL | 权重、用于排序 |

* constraint pubprop_c1 unique (pub_id,type_id,rank)    
* foreign key (pub_id) references pub (pub_id) on delete cascade INITIALLY DEFERRED
* foreign key (type_id) references cvterm (cvterm_id) on delete cascade INITIALLY DEFERRED
* create index pubprop_idx1 on pubprop (pub_id);
* create index pubprop_idx2 on pubprop (type_id);

#### Table `pubauthor_contact`

出版物作者的联系方式

Column | Type | Modifiers | Desc |
---|---|---|---|
pubauthor_contact_id | bigserial | primary key NOT NULL | - |
contact_id | bigint | NOT NULL/foreign key/index | `contact.contact_id` |
pubauthor_id | bigint | NOT NULL/foreign key/index | `pubauthor.pubauthor_id` |

* `CONSTRAINT pubauthor_contact_c1 UNIQUE (contact_id, pubauthor_id)`
* FOREIGN KEY (pubauthor_id) REFERENCES pubauthor(pubauthor_id) ON DELETE CASCADE
* FOREIGN KEY (contact_id) REFERENCES contact(contact_id) ON DELETE CASCADE
* CREATE INDEX pubauthor_contact_idx1 ON pubauthor USING btree (pubauthor_id)
* CREATE INDEX pubauthor_contact_idx2 ON contact USING btree (contact_id)


