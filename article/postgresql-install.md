## postgresql 源码安装

- 下载postgresql源代码

  ```bash
  eastern@eastern-vm:~/Downloads$ wget http://mirrors.ustc.edu.cn/postgresql/source/v13.1/postgresql-13.1.tar.gz
  ```

- 解压

  ```bash
  eastern@eastern-vm:~/Downloads$ tar -zxvf postgresql-13.1.tar.gz
  ```

  ![解压](https://github.com/easternyu/pictures/raw/master/install-postgresql/1-unzip.png)

- 创建目录并修改所属组和属主

  ```bash
  eastern@eastern-vm:~/Downloads$ sudo mkdir -pv /usr/loca/pg131;sudo chown -R eastern:eastern /usr/local/pg131
  ```

  ![创建用户](https://github.com/easternyu/pictures/raw/master/install-postgresql/2-create-dir.png)

- 进入源码目录并开始第一次“configure”

  ```bash
  eastern@eastern-vm:~/Downloads$ cd postgresql-13.1
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ ./confgure --prefix=/usr/local/pg131
  ```

  ![configure](https://github.com/easternyu/pictures/raw/master/install-postgresql/3-configure.png)

- 第一次"configure"报错，提示没有GCC

  ![without-gcc](https://github.com/easternyu/pictures/raw/master/install-postgresql/4-gcc-error.png)

- 安装GCC

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ sudo apt install gcc
  ```

  ![install-gcc](https://github.com/easternyu/pictures/raw/master/install-postgresql/5-install-gcc.png)

- 再次"configure"报错，提示没有readline

  ![without-readline](https://github.com/easternyu/pictures/raw/master/install-postgresql/6-readline-dev-error.png)

- 安装libreadline-dev

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ sudo apt install libreadline-dev -y
  ```

  ![install-readline](https://github.com/easternyu/pictures/raw/master/install-postgresql/7-install-readline-dev.png)

- 再次"configure"报错，提示没有zlib

  ![without-zlib](https://github.com/easternyu/pictures/raw/master/install-postgresql/8-zlib-dev-error.png)

- 安装zlib

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ sudo apt install zlib1g-dev -y
  ```

  ![install-zlib](https://github.com/easternyu/pictures/raw/master/install-postgresql/9-install-zlib-dev.png)

- 再再次"configure"终于没有报错了!

- 开始make，提示没有make

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ make 
  ```

  ![without-make](https://github.com/easternyu/pictures/raw/master/install-postgresql/10-make-and-make-error.png)

- 安装make

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ sudo apt install make -y
  ```

  ![install-make](https://github.com/easternyu/pictures/raw/master/install-postgresql/11-install-make.png)

- make完成并make install

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ make
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ sudo make install
  ```

  ![make-and-makeinstall](https://github.com/easternyu/pictures/raw/master/install-postgresql/12-make-done-and-makeInstall.png)

- 数据库软件安装成功！

  ![install-done](https://github.com/easternyu/pictures/raw/master/install-postgresql/13-soft-install-done.png)

- 创建运行数据库的用户

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ sudo adduser postgres
  ```

  ![create-db-user](https://github.com/easternyu/pictures/raw/master/install-postgresql/14-create-user.png)

- 创建数据目录并且修改所属组和属主

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ sudo mkdir -pv /pgdata;sudo chown -R postgres:postgres /pgdata
  ```

  ![create-data-dir](https://github.com/easternyu/pictures/raw/master/install-postgresql/15-create-data-dir.png)

- 切换至postgres用户

  ```bash
  eastern@eastern-vm:~/Downloads/postgresql-13.1$ su - postgres
  ```

  ![switch-user](https://github.com/easternyu/pictures/raw/master/install-postgresql/16-switch-user.png)

- 添加环境变量

  ```bash
  postgres@eastern-vm:~$ cat << EOF >> .bashrc
  > 
  > export PGDATA=/pgdata
  > export PATH=/usr/local/pg131/bin:$PATH
  > EOF
  postgres@eastern-vm:~$ . .bashrc
  ```

  ![add-env](https://github.com/easternyu/pictures/raw/master/install-postgresql/17-add-env.png)

- 初始化数据库

  ```bash
  postgres@eastern-vm:~$ initdb
  ```

  ![init-db](https://github.com/easternyu/pictures/raw/master/install-postgresql/18-initdb.png)

- 启动数据库

  ```bash
  postgres@eastern-vm:~$ pg_ctl start
  ```

  ![start-db](https://github.com/easternyu/pictures/raw/master/install-postgresql/19-start-db-service.png)

- 创建和进入数据库

  ```bash
  postgres@eastern-vm:~$ createdb test
  postgres@eastern-vm:~$ psql test
  ```

  ![create-db](https://github.com/easternyu/pictures/raw/master/install-postgresql/20-create-db-and-use.png)



> [postgresql documentation](https://www.postgresql.org/docs/current/install-short.html)

