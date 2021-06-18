# MyBatis
### 参考资料
- [MyBatis的工作原理以及核心流程介绍](http://www.mybatis.cn/archives/706.html)

## 工作原理
### SqlSession对象
- 该对象中包含了执行 SQL 语句的所有方法，类似于 JDBC 里面的 Connection；

### Executor接口
- 它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护，类似于JDBC里面的 Statement / PrepareStatement；

### MappedStatement对象
- 该对象是对映射 SQL 的封装，用于存储要映射的 SQL 语句的 id、参数等信息；

### ResultHandler对象
- 用于对返回的结果进行处理，最终得到自己想要的数据格式或类型，可以自定义返回类型。

## 核心流程
![](http://images.intflag.com/mybatis001.jpg)

### 1）读取MyBatis的配置文件
- mybatis-config.xml 为 MyBatis 的全局配置文件，用于配置数据库连接信息；

### 2）加载映射文件
- 映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件 mybatis-config.xml 中加载；
- mybatis-config.xml 文件可以加载多个映射文件，每个文件对应数据库中的一张表；

### 3）构造会话工厂
- 通过 MyBatis 的环境配置信息构建会话工厂 SqlSessionFactory；

### 4）创建会话对象
- 由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法；

### 5）Executor 执行器
- MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护；

### 6）MappedStatement 对象
- 在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息；

### 7）输入参数映射
- 输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型；
- 输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程；

### 输出结果映射
- 输出结果类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型；
- 输出结果映射过程类似于JDBC对结果集的解析过程；

## 多级缓存