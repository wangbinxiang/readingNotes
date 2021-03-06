#4 需要ID

###笔记

---

####建立主键规范

**伪主键,代理键**

大多数表中的每个属性的值都有可能被很多行使用.需要引入一个对于表的领域模型无意义的新列来存储一个伪值.这一列被用作这张表的主键,从而通过它来确定表中的一条记录,即便其他的列允许出现适当的重复项.这种类型的主键列我们通常称为`伪主键`或者`代理键`.

####反模式:以不变应万变

**冗余的键值**

一张表定义了`id`这一列作为主键,然而可能又同时存在另一列上来说更为自然的主键,这一列甚至也具有`UNIQUE`约束.

但是我个人使用观点: 因为我本人使用mysql较多,如果使用innodb引擎,还需要额外考虑如果使用非自增主键,或者没有设定主键但是有其他唯一索引的列(innodb)这样插入效率会非常的低效.具体可以查看`杂技/数据库/索引.md`一章节.所以是否需要设定`id`这一列需要具体场景具体分析.

**允许重复的项**

一个组合键包含了多个不同的列.

		CREATE TABLE BugsProducts (
			id	SERIAL	PRIMARY	KEY,
			bug_id	BIGINT	UNSIGNED NOT NULL,
			product_id	BIGINT	UNSIGNED NOT NULL,
			UNIQUE KEY (bug_id,product_id),
			FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
			FOREIGN KEY (product_id) REFERENCES Products(product_id)
		);
		
当在`bug_id`和`product_id`这两个裂伤应用了唯一性约束,id这一列就会变成多余的.

个人观点:还是和上文一样,具体业务场景具体考虑.

**意义不明的关键字**

`id`这个词是如此地普通,完全无法表达更深层次的意思.

**使用`using`关键字**

如果所有的表都要求定义一个叫做id的伪主键,那么作为外键的列将永远不能使用和引用的列相同的列明.

		...USING (bug_id);
		
		...ON b.id = bp.bug_id
		
**使用组合键之难**

生成唯一键值:

序列通过将运算和实物在逻辑上分离来解决并发问题.序列确保即使在多并发下,每次调用都会返回不同的值,因而无论是否需要将由序列返回的值插入新行,序列都不会再次生成同样的值.由于序列的这种特性,多个客户端可以同时发起请求,并且确信它们获取到的值在每个客户端都是唯一的.

mysql可以通过`LAST_INSERT_ID()`获取一个序列生成的最后一个值.

####如何识别反模式

* 我觉得这张表不需要主键.(开发人员误解了"`主键`"和"`伪主键`"的含义.每张表都必须有一个主键来确保不出现重复项并定位每一行).
* 我怎么能在多对多的表中存储重复的项

####合理使用反模式

使用伪主键,或者通过自动增长的整型机制本身没有什么错误,但不是每张表都需要一个伪主键,更没有必要将每个伪主键都定义成id.

对于太长而不方便实现的自然键来说,伪主键是很好的代替品.

####解决方案:剪裁设计

主键是约束而非数据类型,你可以定义任意列或任意多的列为主键,只要其数据类型支持索引.还可以将一个列的数据类型定义为自增长的整型而不设定其位主键.这两者是`完全无关的`.

个人观点: innodb

如果InnoDB表的数据写入顺序能和B+树索引的叶子节点顺序一致的话,这时候存取效率是最高的，也就是下面这几种情况的存取效率最高:

* 使用自增列(INT/BIGINT类型)做主键,这时候写入顺序是自增的,和B+数叶子节点分裂顺序一致;
* 该表不指定自增列做主键,同时也没有可以被选为主键的唯一索引(上面的条件),这时候InnoDB会选择内置的ROWID作为主键,写入顺序和ROWID增长顺序一致;
* 除此以外，如果一个InnoDB表又没有显示主键,又有可以被选择为主键的唯一索引,但该唯一索引可能不是递增关系时(例如字符串、UUID、多字段联合唯一索引的情况),该表的存取效率就会比较差.

所以具体情况需要具体分析.

**直截了当地描述**

为主键选择更有意义的名称:一个能够反应这个主键锁代表的实体类型的名字.比如,Bugs这张表的主键应该叫做bug_id.


###整理知识点

---