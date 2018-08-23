## python开发：sqlite3

原贴见：[python开发_sqlite3_绝对完整](https://www.cnblogs.com/hongten/p/hongten_python_sqlite3.html)
### SQLite简介
SQLite数据库是一款非常小巧的嵌入式开源数据库软件，也就是说，没有独立的维护进程，所有的维护都来自于程序本身。

在python中，使用sqlite3创建数据库的连接时，当我们指定的数据库文件不存在时，连接对象会自动创建数据库文件；如果数据库文件已经存在，则连接对象不会再创建数据库文件，而是直接打开该数据库文件。

连接对象可以是硬盘上的数据库文件，也可以是建立在内存中，在内存中的数据库执行完任何操作后，都不需要提交事务(commit)

创建在硬盘上面：`conn = sqlite3.connect('c:\\test\\test.db')`
创建在内存上面：`conn = sqlite3.connect('memory:')`

#### 操作
下面以硬盘上面创建数据库文件来具体说明：
```python
conn = sqlite3.connect('c:\\test\\hongten.db')
其中conn对象是数据库链接对象，而对于数据库链接来说，具有以下操作：
* commit()  --- 事务提交
* rollback() --- 事务回滚
* close()    --- 关闭一个数据库链接
* cursor()   --- 创建一个游标
```

**在sqlite3中，所有sql语句的执行都要在游标对象的参与下完成。**
```python
cu = conn.cursor()   #创建了一个游标对象：cu

对于游标对象cu，具有以下具体操作：
* execute()   --- 执行一条sql语句
* executemany()  --- 执行多条sql语句
* close()    --- 游标关闭
* fetchone()  --- 从结果中取出一条记录
* fetchmany()  --- 从结果中取出多条记录
* fetchall()  --- 从结果中取出所有记录
* scroll()   --- 游标滚动
```

