# Hive

## mysql 설치

1. mySQL을 사용하겠다.

   1. 8 버전
   2. https://dev.mysql.com/downloads/file/?id=484922
   3. vi /etc/yum.conf
      1. [main]
         cachedir=/var/cache/yum/$basearch/$releasever
         keepcache=0
         debuglevel=2
         logfile=/var/log/yum.log
         exactarch=1
         obsoletes=1
         **gpgcheck=0**  -> gpg전자서명 체크 부분인데 0으로 했다가 설치 끝나면 1로 다시 바꿔줌.
         plugins=1
         installonly_limit=5
         bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
         distroverpkg=centos-release

   3.  rpm 실행하기 
      1. rpm -ivh mysql80-community-release-el7-3.noarch.rpm

~~~ 
경고: mysql80-community-release-el7-3.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
준비 중...                         ################################# [100%]
Updating / installing...
   1:mysql80-community-release-el7-3  ################################# [100%]
~~~

  4. yum install -y mysql-server

       1. Installed:
            mysql-community-server.x86_64 0:8.0.21-1.el7

          Dependency Installed:
            mysql-community-client.x86_64 0:8.0.21-1.el7        mysql-community-common.x86_64 0:8.0.21-1.el7
            mysql-community-libs.x86_64 0:8.0.21-1.el7

          Complete!

          		2. **gpgcheck=1** -> gpg전자서명 체크 부분인데 0으로 했다가 설치 끝나면 1로 다시 바꿔줌.

		5. mySQL을 리눅스의 시작프로그램으로 등록하고 싶다. 

              		1.  systemctl enable mysqld
           systemctl enable mysql-daemon
              		2. 만약 등록하지 않았다면 systemctl start mysqld을 해주면 된다. 

		6. 설치 및 실행 끝

- 그런데 실행을 하면 비밀번호를 치라고 한다. 

- 그래서 로그를 보면 임시 비밀번호가 있다. 

~~~
cat /var/log/mysqld.log
===================================================
2020-08-25T06:31:10.654023Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.21) initializing of server in progress as process 11005
2020-08-25T06:31:10.662521Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-08-25T06:31:12.105521Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-08-25T06:31:13.331342Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: R!0wa14xg_a2
2020-08-25T06:31:16.701580Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.21) starting as process 11051
2020-08-25T06:31:16.720084Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-08-25T06:31:17.539018Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-08-25T06:31:17.696227Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2020-08-25T06:31:18.107955Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-08-25T06:31:18.108152Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2020-08-25T06:31:18.131431Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.21'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server - GPL.
~~~

- 비밀번호가 있고 이제 그걸로 로그인을 하면 된다. 

~~~
mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.21

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
~~~

- 비밀번호가 어려우니 바꾸자.

~~~ sql
alter user 'root'@'localhost' Identified by '178432';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
~~~

- 안먹힌다

~~~sql
show variables like 'validate_password%';
~~~

- 이것도 안먹힌다.

~~~sql
mysql> alter user 'root'@'localhost' identified by '@Gur178432';
Query OK, 0 rows affected (0.01 sec)
~~~

- 이렇게 해서 일단 임시 비밀번호를 바꿔 놓는다.
- 그리고 패스워드의 규칙을 보자.

~~~ sql
mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)
~~~

- 조작해서 쉽게 바꿔보자.

~~~ sql
mysql> set GLOBAL validate_password.length=4;
Query OK, 0 rows affected (0.00 sec)

mysql> set GLOBAL validate_password.mixed_case_count=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set GLOBAL validate_password.policy=LOW;
Query OK, 0 rows affected (0.00 sec)

mysql> set GLOBAL validate_password.special_char_count=0;
Query OK, 0 rows affected (0.00 sec)

mysql> alter user 'root'@'localhost' identified by '178432';
Query OK, 0 rows affected (0.01 sec)
~~~

- 정책을 바꿔서 원하는 비밀 번호로 한다. 



### Hive에서 사용할 DB를 만들어 보자. (메타 정보용)

~~~ sql
mysql> create database hive_db default character set utf8;
Query OK, 1 row affected, 1 warning (0.01 sec)
~~~

- hive_db라는 db를 만듬.

~~~ sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| hive_db            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
~~~



### 루트에게 권한을 주자.

- 그냥은 유저도 못만들고 권한도 못준다.

~~~ sql
mysql> CREATE USER 'root'@'%' IDENTIFIED BY '1234';
Query OK, 0 rows affected (0.01 sec)
~~~

- 8버전 부터는 다른 계정을 하나 만들고 해야한다.

~~~sql
mysql> grant all privileges on *.* to 'root'@'%' with grant option;
Query OK, 0 rows affected (0.01 sec)
~~~

- 그다음 권한을 줘보자.
- 혹시 버퍼에 남아있을지 모르니 플러쉬해준다.

~~~sql
flush privileges;
~~~



### Hive가 쓸 유저를 만들자.

~~~sql
mysql> create user 'hiveuser'@'%' identified by '178432';
Query OK, 0 rows affected (0.01 sec)
~~~

- 그리고 만든 유저에 hive_db의 권한을 모두 준다.

~~~sql
mysql> grant all privileges on hive_db.* to hiveuser@'%' ;
Query OK, 0 rows affected (0.01 sec)
~~~





## Hive 설치

- https://hive.apache.org/downloads.html
- 현재 사용하고 있는 하둡의 버전과 잘 맞춰야하기 때문에 2.3.7로 받았다.
- http://apache.mirror.cdnetworks.com/hive/hive-2.3.7/
- 압축을 푼다.



#### 하이브의 conf폴더에 있는 template을 하둡 때처럼 수정해 줄것이다.

- 기본적으로 이름을 바꿔주었다.

1. hive-env.sh

   1. HADOOP_HOME을 하둡 설치한 위치로 잡는다.
   2. HADOOP_HOME=/home/companion_tazo/hadoop

2. hive-default.xml

   1. WARNING은 건드리지 않기

   2. 하이브에서 mysql db접속할 때 쓰는 db URL 포트 설정.

   3. 하이브 메타스토어 웨어 하우스 위에 추가하기

      1.   <property>
             <name>hive.metastore.local</name>
             <value>false</value>
           </property>

         

   4.   이 부분을 교체 해줄 것이다.

      1. <property>
             <name>javax.jdo.option.ConnectionURL</name>
             <value>jdbc:derby:;databaseName=metastore_db;create=true</value>
             <description>
               JDBC connect string for a JDBC metastore.
               To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
               For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
             </description>
           </property>

      2.   이렇게

         <property>
             <name>javax.jdo.option.ConnectionURL</name>
             <value>jdbc:mysql://localhost:3306/hive_db?createDatabaseIfNotExist=true</value>
             <description>
               JDBC connect string for a JDBC metastore.
               To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
               For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
             </description>
           </property> 

      3. 포트 확인법  netstart -tlpn | grep mysqld

         

   5. javax.jdo.option.ConnectionDriverName

      1.   <property>
             <name>javax.jdo.option.ConnectionDriverName</name>
             <value>org.apache.derby.jdbc.EmbeddedDriver</value>
             <description>Driver class name for a JDBC metastore</description>
           </property>

      2.   <property>
             <name>javax.jdo.option.ConnectionDriverName</name>
             <value>com.mysql.jdbc.Driver</value>
             <description>Driver class name for a JDBC metastore</description>
           </property>

         

   6. javax.jdo.option.ConnectionUserName

      1.   <property>
             <name>javax.jdo.option.ConnectionUserName</name>
             <value>APP</value>
             <description>Username to use against metastore database</description>
           </property>
           
      2.   <property>
             <name>javax.jdo.option.ConnectionUserName</name>
             <value>hiveuser</value>
             <description>Username to use against metastore database</description>
           </property>
           
           
      
      7. javax.jdo.option.ConnectionPassword

            1. mine >> 178432

            2. <property>
                 <name>javax.jdo.option.ConnectionPassword</name>
                 <value>mine</value>
                 <description>password to use against metastore database</description>
               </property>

            3. <property>
                 <name>javax.jdo.option.ConnectionPassword</name>
                 <value>178432</value>
                 <description>password to use against metastore database</description>
               </property>

               

      8. hive.cli.print.header

            1. false >> true
            2. <property>
                 <name>hive.cli.print.header</name>
                 <value>false</value>
                 <description>Whether to print the names of the columns in query output.</description>
               </property>
            3. <property>
                 <name>hive.cli.print.header</name>
                 <value>true</value>
                 <description>Whether to print the names of the columns in query output.</description>
               </property>

   9. JDBS 설치를 해야한다.

      1. mysql서버에 가서 Connector/J 8.0.21을 플랫폼에 상관없이 받는다.

      2. 압축을 풀고 mysql-connector-java-8.0.21.jar 복사하여 /mnt/Share에 넣는다.

      3. 그 파일을 아파치 lib에 옮긴다.

         

   10. 하둡에서 Hive를 실행하면 임시 파일을 저장하는 공간을 만들어 줘야한다.

       1. bin/hdfs /tmp/(hive를 실행할계정, 나같은 경우는 companion_tazo)

       2. chmod로 tmp는 775, 실행할 계정은 777이 되어야 한다. 

          

   11. Hive가 문제가 있을 때 metastore를 초기화 시도해본다.

       1. bin/schematool -initSchema -dbType mysql
       2. 그런데 타임존이 달라 에러가 난다.
       3. mysql은 UTC로 되어있기 때문에 그런거 같다.
       4. 타임존을 맞춰보자

       

   12.  sudo vi /etc/my.cnf

       1. default-time-zone='+9:00' 을 추가해주고 나온다.

       2. 기본은 UTC로 되어있어서 한국 시간으로 맞춰줬다.

       3. systemctl stop mysqld >> systemctl stop mysqld

          

   13. vi conf/hive-site.xml

       1. 추가해주자.
       2. 그냥 bin/hive를 실행했는데 java에러가 떠서 스택 오버 플로우에서 저렇게 추가해주면 된단다.
       3.   <property>
              <name>system:java.io.tmpdir</name>
              <value>/tmp/companion_tazo</value>
            </property>

   14. bin/hive 실행

       

   


