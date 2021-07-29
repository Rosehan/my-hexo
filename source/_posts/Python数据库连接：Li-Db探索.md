---
title: Python数据库连接：Li_Db探索
author: Lilihx
tags:
  - Python
categories:
  - code
date: 2021-01-26 13:01:00
---
## 前言
>在日常编写Python脚本的时候，总会遇到一种头疼的情况：拼接SQL。Pymysql，Mysql-connector等第三方数据库连接工具都只是提供了最简单的execute操作，增删改查语句都需要我们手拼SQL，这在日常开发中是极其不便的。

>此外，成熟的Server开发语言如Java，PHP等，都会将数据库连接生成配置文件单独维护。因此基于上面两种痛点，自己在PymySQL上封装了一层，来解决这两个问题。

<!--more-->

## 1

Li_Db的简单类图如下：

![截屏2021-01-26 下午1.21.00](http://bj.bcebos.com/ibox-thumbnail98/ae57f990546710071579eb8f5b05a5a6?authorization=bce-auth-v1%252Ffbe74140929444858491fbf2b6bc0935%252F2021-01-26T05%253A21%253A04Z%252F1800%252F%252F066b9c87f27dad8b5a19a5ef5f1580cc046e75275ff42a6954fcdb8dc6d39a12)



一个上层调用者，不再需要导入PyMysql，只需要实例化一个Db_Mgr，然后配置响应的config_file即可。

一个Format_insert语句如下所示：

```Python
def format_insert(table, row):
    """DESC: 构造sql语句 
    Arguments：
    table：表名 
    row  ：待插入数据，字典型 
    """
    sql = "INSERT INTO %s ( " % table 

    for key in row.keys():
        sql += "`%s`" % key 
        sql += ','
    # 去除最后一个逗号
    sql = sql[:-1]
    sql += ') VALUES ('

    for val in row.values():
        # val 是int or float 不用加引号
        if type(val) == int or type(val) == float:
            sql += str(val)
            sql += ','
            continue 
        sql += ("\"%s\"," % val)
    # 去除最后一个,
    sql = sql[:-1]
    sql += ')'     
    return sql 
 
```

其余的format思想也都类似，调用者只需要传入key-value，Li_Db便可自动将其生成SQL，并执行。

Db_Mgr代码示例如下所示：
```Python
class DBMgr(object):
    """数据库管理类
    """
    db = ''
    def __init__(self, clustername):
        """构造函数

        Args:
            clustername ([type]): [description]
        """
        self.connect(clustername)

    def connect(self, clustername):
        """读取cluster配置文件

        Args:
            clustername ([type]): [description]
        """
        
        # 首先读取cluster配置文件
        config_filename = "db.ini"

        now_path = os.path.join(os.path.dirname(__file__))
        config_file = os.path.join(now_path, config_filename)
        cf = configparser.ConfigParser()
        cf.read(config_file)

        ip = cf.get(clustername, "ip")
        username = cf.get(clustername, "username")
        password = cf.get(clustername, "password")
        db = cf.get(clustername, "db")
        port = cf.getint(clustername, "port")

        self.get_conmgr(ip, username, password, db, port)

    def get_conmgr(self, host, user, passwd, db='', port=3306,
                charset='utf8', log_name=None, log_file=None):
        """连接数据库

        Args:
            host ([type]): [description]
            user ([type]): [description]
            passwd ([type]): [description]
            db (str, optional): [description]. Defaults to ''.
            port (int, optional): [description]. Defaults to 3306.
            charset (str, optional): [description]. Defaults to 'utf8'.
            log_name ([type], optional): [description]. Defaults to None.
            log_file ([type], optional): [description]. Defaults to None.
        """
        try:
            self.conn = pymysql.connect(host=host, user=user, 
            passwd=passwd, port=port, read_timeout=30, charset=charset)
            self.cur = self.conn.cursor(cursor=pymysql.cursors.DictCursor)
            if self.is_db_exist(db):
                self.cur.execute('USE %s' % db)
                print('select database success. db: %s' % db)
                self.db = db
            else:
                self.select_db(db)
            self.cur.execute('SET NAMES %s' % charset)
        except pymysql.Error as e:
            print('Mysql Error: %s' % (e))
            self.cur = False
        

    def is_db_exist(self, db):
        """ 数据库是否存在
            Args:
            db: 数据库名称
        Return:
            True: 存在
            False: 不存在
        """
        table = 'information_schema.SCHEMATA'
        columns = '*'
        param = 'WHERE SCHEMA_NAME=\'%s\'' % db
        try:
            result = self.query(table, columns, param)
            if result:
                return True
            return False
        except pymysql.Error as e:
            print('Mysql Error: %s' % (e))
            return False
    
    def execute(self, sql):
        """直接执行SQL语句

        Args:
            sql ([type]): [description]
        """
        try:
            self.cur.execute(sql)
            self.conn.commit()
            result = self.cur.fetchall()
            return result
        except pymysql.Error as e:
            print("Mysql Error: %s" % e)
            return False

    def query(self, table, columns, param):
        """查询数据库
        Args:
            table: 表名
            columns: 插入的列
            param: 查询条件
        Return:
            Dict: 包含查询结果的字典
            False: 查询失败
        """
        sql = 'SELECT %s FROM %s %s' % (columns, table, param)
        print('sql: %s' % sql)
        try:
            start_time = time.time()
            self.cur.execute(sql)
            end_time = time.time()
            print('execute sql cost: %f' % (end_time - start_time))
            start_time = time.time()
            result = self.cur.fetchall()
            end_time = time.time()
            print('fetch result cost: %f' % (end_time - start_time))
            resultdict = []
            start_time = time.time()
            for item in result:
                resultdict.append(item)
            end_time = time.time()
            print('append result cost: %f' % (end_time - start_time))
            return resultdict
        except pymysql.Error as e:
            print('Mysql Error: %s' % e)
            return False
```

而Db_Mgr可以封装自定义你想要的一切SQL操作，比如getById, multiInsert等等。这样就能在多个Python脚本之中复用了。



## 2，可优化

Format_SQL只支持最简单的 fileds 和 where语句，更复杂的条件查询，聚合函数等等复杂的SQL，需要优化FormatSQL。我见过一个PHP版本的类似操作，整个文件达到了大几百行，因此实现起来还是很复杂的。

希望以后能逐步完善。