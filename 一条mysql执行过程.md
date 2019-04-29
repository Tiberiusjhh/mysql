我们都知道如何使用mysql语句，但是我们并不清楚一条mysql语句的具体流程是如何执行的？
首先下面有一张mysql语句执行流程图可以帮我们理解执行的顺序(图片是借鉴另外一个git作者的图片，本来想自己画图的，但是用Xmind好像画不出这样的，有没有推荐Mac好用的画图软件，谢谢)
由图中我们得知几个关键的步骤：<br>
**连接器：** 连接器的作用就是我们在客户端中登陆Mysql的时候对身份和角色权限验证。<br>
**查询器：**（注⚠️：Mysql在8.0以前是会查询到数据缓存，8.0以后就被删除了）执行查询语句时，会先去缓存中查询，若是没有才会进行后续查询，并将查询到的数据缓存<br>
**分析器：**对SQL语句进行分析，提取关键字如（select * from stduent where age = ‘12’ and name=‘如画’）语句中关键字是select、student（表名）、条件语句<br>
然后才会进行语句合法性的检验。随后才会进行下一步优化器<br>
**优化器：** 优化器自己来决定最优的执行方案（但是这个最优执行方案并不一定就是最优的），然后这个时候SQL语句执行的方案就已经确定了<br>
**执行器：** 到了执行器以后，首先要检查执行语句的权限，若是没有对应执行权限那么返回错误信息，如果有权限，就调用引擎接口，返回执行结果数据。<br>

![image](https://github.com/Tiberiusjhh/mysql/blob/master/assets/mysql.jpg)


简单来说 MySQL 主要分为 Server 层和存储引擎层：<br>

**Server 层：** 主要包括连接器、查询缓存、分析器、优化器、执行器等，所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图，函数等，还有一个通用的日志模块 binglog 日志模块。<br>
**存储引擎：** 主要负责数据的存储和读取，采用可以替换的插件式架构，支持 InnoDB、MyISAM、Memory 等多个存储引擎，其中 InnoDB 引擎有自有的日志模块 redolog 模块。现在最常用的存储引擎是 InnoDB，它从 MySQL 5.5.5 版本开始就被当做默认存储引擎了。<br>

* Red * Red * Red * Red<br>
* Red * Red * Red * Red<br>
* Red * Red * Red * Red<br>
* Red * Red * Red * Red<br>
* Red * Red * Red * Red<br>
* Red * Red * Red * Red<br>
