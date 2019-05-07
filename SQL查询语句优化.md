Mysql查询语句优化

1.在使用索引的where查询语句中，尽量不使用!=或<>来进行查询，否则引擎将抛弃索引从而进行全表扫描

2.查询中尽量避免全表扫描查询，在where或者order by中建立索引查询

3.避免在where语句中进行null查询，null查询会使引擎放弃索引从而进行全表扫描查询
例：
select id,name from user where num is null;
改进:我们可以将num默认设置为0来确保表中没有null
select id,name from user where num = 0;

4.尽量避免在where查询语句中使用or语句，in语句也会使引擎放弃索引。可以使用union all替换
select id from user where num = 20 or num = 10;
替换:
select id from user where num =10
union all 
select id from user where num =10;
注意：union 和 union all 区别
两个使用都要有相同数量的列，列也必须要相同的数据类型，同时每条select语句中的列顺序也必须相同
union：是两个表数据的并集（不会有重复相同的数据）
union all：是两个表的集合（会有数据重复）
http://www.w3school.com.cn/sql/sql_union.asp

5.下面的查询也会导致全表扫描（%不可前置）
select id,name from user name like '�c%';
若是想提高效率可以考虑全文检索

6.in 和 not in 也会导致全表扫描
select id,name from user where num in (1,2,3);
但是我们可以使用between来替换避免出现全表扫描,但是要求其中的查询条件是连续性
select id,name from user where num between 1 and 3;

7.如果在where语句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然而，如果自编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描
select id,name from user where num = @num;
可以改为强制查询使用索引
select id,name from user with(index(索引名)) where num =@num;

8.在where语句中尽量避免对字段使用表达式，这会使查询放弃引擎
select id,name from user where num/2=20;
可改进为:
select id,name from user where num = 2*20;

9.不要在where语句使用过程中的"="的左边使用函数、算数运算、以及其他表达式，这也会使得查询放弃索引查询。

10.在使用索引字段作为条件时，如果该索引时复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。

11.不要写一些没有意义的查询，例如需要生成一个空表
select id,name into #t from user where 1= 0;
这类代码不会生成任何结果，但是却会消耗系统资源，应该改成如下：
create table #t(……)

12.很多时候exists代替in使用效果会更好
select id,name from user where num id(select num from user2 );
代替如下：
select id,name from user where num exists(select 1 from user2 where num = user.num);
exists和in使用区别：子查询的表得出的结果小，主表大而且还有索引的情况下使用in
               相反子查询的表得出的结果大，主表小而且还有索引的情况下使用exists
https://www.cnblogs.com/emilyyoucan/p/7833769.html

13.并不是所有的索引对查询都有效，SQL时根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，例如sex、male、female几乎各占一半时，在sex上建立索引对查询效率也不会起作用的

14.索引并不是越多越好，索引固然能提高select的查询效率，但是同时也会降低insert和update的效率，因为insert和update可能会引起索引的重建。怎样建索引我们需要根据业务需要慎重考虑，并且一个表的索引最好不要超过6个，若是超过了那么就需要考虑一些不常用的列上建立索引是否有必要。

15.尽可能避免更新 clustered （聚簇索引）索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理储存顺序，一旦该列值改变将导致整个表记录的顺序的调整，会消耗相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将索引键为 clustered 索引。

16.尽量使用数值类型字段，若是字段类型全是数字那么请使用数值类型，不要设计为字符型，这会降低查询和连接性能，并且会增加储存开销。这是因为引擎在处理查询和连接的时候 会逐个比较字符型中的每一个字符，而数值类型只需要对比一次（类似于Java中数值比较使用== 与 对比字符串使用equals的区别）

17.尽可能使用varchar/nvarchar代替cahr/nchar，因为首先变长字段储存空间小，可以节省储存空间，其次对于查询来说在一个相对小的字段内搜索效率会比较高
varchar/Nvarchar区别:
varchar(n)
长度为 n 个字节的可变长度且非 Unicode 的字符数据。n 必须是一个介于 1 和 8,000 之间的数值。存储大小为输入数据的字节的实际长度，而不是 n 个字节。
nvarchar(n)
包含 n 个字符的可变长度 Unicode 字符数据。n 的值必须介于 1 与 4,000 之间。字节的存储大小是所输入字符个数的两倍。

18.任何时候都不要使用‘select * from user’查询语句，请将‘*’替换为具体业务所需的字段，不要返回任何用不到的多余字段。（适用于任何下mysql）

19.尽量使用变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。

20.避免频繁创建和删除临死表，以减少系统资源的消耗。

21.临时表并不是不可使用，适当的使用他们可以使某些例程更有效，例如，当需要重读引用大型表或常用表中的某个数据集时。但是，对于一次性事件，最好使用 导出表

22.在新建临时表时，如果一次性插入数据量很大，那么可以使用select into代替 create table，避免造成大量log，以提高速度；如果数据量不大，为来缓和系统表的资源，应先create table，然后再insert

23.如果使用到了临时表，在储存过程的最后务必将所有的临时表显示删除，先truncate table，然后drop table，这样可以避免系统表的较长时间锁定。

24.尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1w行，那么就应该考虑改写
什么是游标：https://blog.csdn.net/next_sun/article/details/7447687

25.使用基于游标的方法或临时表方法之前，应先找基于集的解决方案来解决问题，基于集的方法通常更有效。

26.与临时表一样，游标并不是不可使用。对小型数据集使用FAST_FORWARD游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括‘合计’的例程通常比较使用游标执行的速度快。如果开发时间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。
游标：Static|Keyset|DUNAMIC|FAST_FORWARD
Static 选项
Static选项相当于从tempdb里面完全缓存一个结果集。外部修改数据，并不影响到游标本身(修改游标结果集任意一列都不影响)。使用Static选项的话，不能执行更新游标的 Current of 操作
PS：就是说你在执行这段代码的时候，另外一个窗口即时插入新数据，修改数据，删除数据也不会影响到当前游标

 
Keyset 选项
Keyset 选项也是从tempdb里面缓存一个结果集，只缓存一个主键。外部修改数据，不能修改主键，修改其它列是有效的。如果基表该行被删除了，@@Fetch_State返回值为-2
PS：就是说你在执行这段代码的时候，另外一个窗口即时插入新数据没有影响。修改非主键数据可以获取到，如果数据不存在就88啦


DYNAMIC 选项
每次获取都即时更新，新增，修改，删除都可以支持。动态游标不支持 ABSOLUTE 提取选项

FAST_FORWARD
指定启用了性能优化的 FORWARD_ONLY、READ_ONLY 游标。如果指定了 SCROLL 或 FOR_UPDATE，则不能也指定 FAST_FORWARD。


27.在所有的储存过程和触发器的开始设置SET NOCOUNT ON，在结束时设置SET NOCOUNT OFF。无需在执行储存过程和触发器的每个语句后向客户端发送DONE_PROC消息。

28.尽量避免向客户端返回大量数据，如是数据量大，那么需要考虑相应需求是否合理。

29.尽量避免大量事务操作，提高系统并发能力。
