## postgresql基本使用--数据库的基本操作

* 创建数据库

  * 基本语法

    ```sql
    CREATE DATABASE name
    	[ [ WITH ] [ OWNER [=] user_name]
        	[ TEMPLATE [=] template]
        	[ ENCODING [=] encoding]
        	[ LC_COLLATE [=] lc_collate]
        	[ LC_CTYPE] [=] lc_ctype]
        	[ TABLESPACE [=] tablespace]
        	[ CONNECTION LIMIT [=] connlimit ] ]
    ```

    参数说明：

    ​	OWNER：指定数据库属于哪个用户

    ​	TEMPLATE：从哪个数据库模板创建

    ​	ENCODING：使用的字符编码，注意：postgresql没有gbk编码，一般使用utf8即可

    ​	TEBLESPACE：指定表空间

    ​	CONNECTION LIMIT：限制连接数量

  * 一般创建数据库不需要这么麻烦的语法，最简单的创建语句如下：

    ```sql
    CREATE DATABASE testdb01;
    ```

* 修改数据库属性

  * 基本语法

    ```sql
    ALTER DATABASE name [ [ WITH ] option [ ... ] ]
    ```

    option可以是：

    ​	CONNECTION LIMIT connlimit：修改连接数限制

    ​	ALTER DATABASE name RENAME TO new_name：修改数据库名

    ​	ALTER DATABASE name OWNER TO new_owner：修改数据库所属用户

    ​	ALTER DATABASE name SET TABLESPACE new_tablespace：修改数据库表空间

    ​	ALTER DATABASE name SET configuration_parameter {TO|=} {value|DEFAULT}：修改参数为其他值或默认值

    ​	ALTER DATABASE name SET configuration_parameter FROM CURRENT：修改参数为当前值

    ​	ALTER DATABASE name RESET configuration_parameter：重置参数

    ​	ALTER DATABASE name RESET ALL：重置所有参数

  * 修改示例：

    * 修改数据库最大连接数为20

      ```sql
      test=# alter database testdb01 connection limit 20;
      ```

    * 修改数据库名为testdb02

      ```sql
      test=# alter database testdb01 rename to testdb;
      ```

    * 关闭数据库默认索引扫描

      ```sql
      test=# alter database testdb set enable_indexscan to off;
      ```

* 删除数据库

  * 基本语法

    ```sql
    DROP DATABASE [ IF EXISTS ] name;
    ```

  * 删除数据库示例

    * 删除数据库

      ```sql
      test=# drop database testdb;
      ```

    * 如果数据库存在删除，否则跳过

      ```sql
      test=# drop database if exists testdb;
      ```

      ![drop-not-exists-db](https://github.com/easternyu/pictures/raw/master/pg-database/drop-not-exists-db.png)

    * 注意，如果还有用户连接在这个数据库上，则不能删除该数据库
    
      ![drop-open-db](https://github.com/easternyu/pictures/raw/master/pg-database/drop-open-db.png)
    
    