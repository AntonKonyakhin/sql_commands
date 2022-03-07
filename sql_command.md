### Задача 1  
Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.
```
root@anton-v-m:~/lesson7/posgresql# docker run --name posgresdb -v /opt/postgree_backup/ -v /var/postgresqldb/ -e POSTGRES_PASSWORD=Netology -p 5432:5432 -d bitnami/postgresql:12.10.0
8b1f40aac1d9bb4c1178877dfde7da1b25aa68828dda4148e90d6856978d6099
root@anton-v-m:~/lesson7/posgresql# docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS         PORTS                                       NAMES
8b1f40aac1d9   bitnami/postgresql:12.10.0   "/opt/bitnami/script…"   9 seconds ago   Up 7 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   posgresdb
root@anton-v-m:~/lesson7/posgresql# 
```

### Задача 2
В БД из задачи 1:

создайте пользователя test-admin-user и БД test_db
в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
создайте пользователя test-simple-user
предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db
Таблица orders:

id (serial primary key)
наименование (string)
цена (integer)
Таблица clients:

id (serial primary key)
фамилия (string)
страна проживания (string, index)
заказ (foreign key orders)
Приведите:

итоговый список БД после выполнения пунктов выше,
описание таблиц (describe)
SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
список пользователей с правами над таблицами test_db

подключаемся к контейнеру

```
docker exec -it posgresdb /bin/bash
```

подключаемся к postgresql
```
psql -U postgres -h localhost
```

создаем пользователя и БД
test-admin-user и БД test_db
```
postgres=# CREATE DATABASE test_db;
CREATE DATABASE

```

```
postgres=# CREATE USER "test-admin-user" WITH ENCRYPTED PASSWORD 'Netology';
CREATE ROLE
postgres=# \du
                                      List of roles
    Role name    |                         Attributes                         | Member of 
-----------------+------------------------------------------------------------+-----------
 postgres        | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test-admin-user |                                                            | {}

```
проверяем базы данных
```
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(4 rows)

postgres=# \q
```
создайте таблицу orders и clients (спeцификация таблиц ниже)
```
test_db=# create table orders (id SERIAL PRIMARY KEY, наименование text, цена INT);
CREATE TABLE
```
```
test_db=# create table clients (id SERIAL PRIMARY KEY, фамилия text, "страна проживания" text, заказ INT, foreign key(заказ) references orders(id));
CREATE TABLE

```
предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
```
test_db=# grant ALL on ALL tables in schema public to "test-admin-user";
GRANT
```
создайте пользователя test-simple-user
```
postgres=# create user "test-simple-user" with encrypted password 'Netology';
CREATE ROLE

```
предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db
```
test_db=# grant SELECT, INSERT, UPDATE, DELETE on all tables in schema public to "test-simple-user";
GRANT
```

итоговый список БД после выполнения пунктов выше
```
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 

```

описание таблиц (describe)
```
test_db=# \d
               List of relations
 Schema |      Name      |   Type   |  Owner   
--------+----------------+----------+----------
 public | clients        | table    | postgres
 public | clients_id_seq | sequence | postgres
 public | orders         | table    | postgres
 public | orders_id_seq  | sequence | postgres
(4 rows)


test_db=# \d orders
                               Table "public.orders"
    Column    |  Type   | Collation | Nullable |              Default               
--------------+---------+-----------+----------+------------------------------------
 id           | integer |           | not null | nextval('orders_id_seq'::regclass)
 наименование | text    |           |          | 
 цена         | integer |           |          | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)

test_db=# \d clients
                                  Table "public.clients"
      Column       |  Type   | Collation | Nullable |               Default               
-------------------+---------+-----------+----------+-------------------------------------
 id                | integer |           | not null | nextval('clients_id_seq'::regclass)
 фамилия           | text    |           |          | 
 страна проживания | text    |           |          | 
 заказ             | integer |           |          | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "indcountry" btree ("страна проживания")
Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)


```
SQL-запрос для выдачи списка пользователей с правами над таблицами test_db

```
SELECT * from information_schema.table_privileges where table_catalog = 'test_db';
```

список пользователей с правами над таблицами test_db
```
test_db=# \dp
                                           Access privileges
 Schema |      Name      |   Type   |         Access privileges          | Column privileges | Policies 
--------+----------------+----------+------------------------------------+-------------------+----------
 public | clients        | table    | postgres=arwdDxt/postgres         +|                   | 
        |                |          | "test-admin-user"=arwdDxt/postgres+|                   | 
        |                |          | "test-simple-user"=arwd/postgres   |                   | 
 public | clients_id_seq | sequence |                                    |                   | 
 public | orders         | table    | postgres=arwdDxt/postgres         +|                   | 
        |                |          | "test-admin-user"=arwdDxt/postgres+|                   | 
        |                |          | "test-simple-user"=arwd/postgres   |                   | 
 public | orders_id_seq  | sequence |                                    |                   | 
(4 rows)


```

### Задача 3  
Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:  
```
test_db=# insert into orders (наименование, цена) values ('Шоколад', 10), ('Принтер', 3000), ('Книга', 500), ('Монитор', 7000), ('Гитара', 4000);
INSERT 0 5
test_db=# select * from orders;
 id | наименование | цена 
----+--------------+------
  4 | Шоколад      |   10
  5 | Принтер      | 3000
  6 | Книга        |  500
  7 | Монитор      | 7000
  8 | Гитара       | 4000
(5 rows)

```

```
test_db=# insert INTO clients (фамилия, "страна проживания") Values ('Иванов Иван Иванович', 'USA'), ('Петров Петр Петрович', 'Canada'), ('Иоганн Себастьян Бах', 'Japan'), ('Ронни Джеймс Дио', 'Russia'), ('Ritchie Blackmore','Russia');
INSERT 0 5
test_db=# 
test_db=# 
test_db=# select * from clients;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  2 | Иванов Иван Иванович | USA               |      
  3 | Петров Петр Петрович | Canada            |      
  4 | Иоганн Себастьян Бах | Japan             |      
  5 | Ронни Джеймс Дио     | Russia            |      
  6 | Ritchie Blackmore    | Russia            |      
(5 rows)


```

Используя SQL синтаксис:  

вычислите количество записей для каждой таблиц
приведите в ответе:  
запросы
результаты их выполнения.
```
test_db=# select count(*) from clients;
 count 
    5
(1 row)

test_db=# select count(*) from orders;
 count 
     5
(1 row)

```

### Задача 4
Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

```
test_db=# select * from clients;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  2 | Иванов Иван Иванович | USA               |      
  3 | Петров Петр Петрович | Canada            |      
  4 | Иоганн Себастьян Бах | Japan             |      
  5 | Ронни Джеймс Дио     | Russia            |      
  6 | Ritchie Blackmore    | Russia            |      
(5 rows)

test_db=# select * from orders;
 id | наименование | цена 
----+--------------+------
  4 | Шоколад      |   10
  5 | Принтер      | 3000
  6 | Книга        |  500
  7 | Монитор      | 7000
  8 | Гитара       | 4000
(5 rows)

test_db=# update clients set заказ=6 where id=2;
UPDATE 1
test_db=# update clients set заказ=7 where id=3;
UPDATE 1
test_db=# update clients set заказ=8 where id=4;
UPDATE 1
test_db=# select * from clients;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  5 | Ронни Джеймс Дио     | Russia            |      
  6 | Ritchie Blackmore    | Russia            |      
  2 | Иванов Иван Иванович | USA               |     6
  3 | Петров Петр Петрович | Canada            |     7
  4 | Иоганн Себастьян Бах | Japan             |     8
(5 rows)

```

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
```
test_db=# select фамилия from clients where заказ is not null;
       фамилия        
----------------------
 Иванов Иван Иванович
 Петров Петр Петрович
 Иоганн Себастьян Бах
(3 rows)

```

### Задача 5  
Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

```
test_db=# explain select фамилия from clients where заказ is not null;
                        QUERY PLAN                         
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=32)
   Filter: ("заказ" IS NOT NULL)
(2 rows)


```
Последовательно сканирование таблицы clients с фильтром   
приблизительная стоимость запуска 0.00  
приблизительная стоимость выполнения запроса 18.30  
ожидаемое число строк 806  
ожидаемый размер строк 32


### Задача 6 (в лекции  говорили, что можно не делать 6 задание, так как еще не проходили postgresql, но нагуглил, ничего сложного :))  

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления.


бэкап базы на volume
```
root@eb4854a2ba57:/opt# pg_dump -U postgres -W test_db > /opt/postgres_backup/test_db.backup
```
остановил контейнер
```
root@ubuntukurs:~# docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED        STATUS         PORTS                                       NAMES
eb4854a2ba57   bitnami/postgresql:12.10.0   "/opt/bitnami/script…"   28 hours ago   Up 6 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgresdb
root@ubuntukurs:~# docker stop postgresdb 
postgresdb
root@ubuntukurs:~# docker ps -a
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS                      PORTS     NAMES
bde54ad1a8ab   newpostgres                  "/opt/bitnami/script…"   17 minutes ago   Exited (0) 7 minutes ago              postgres_new
eb4854a2ba57   bitnami/postgresql:12.10.0   "/opt/bitnami/script…"   28 hours ago     Exited (0) 21 seconds ago             postgresdb
74f968ef4924   debian                       "echo 'Hello world'"     28 hours ago     Exited (0) 28 hours ago               eager_bell

```
запустил новый контейнер
```
root@ubuntukurs:~# docker images
REPOSITORY           TAG       IMAGE ID       CREATED          SIZE
newpostgres          latest    81eea1ee3806   21 minutes ago   269MB
bitnami/postgresql   12.10.0   abb450842889   35 hours ago     269MB
debian               latest    d40157244907   6 days ago       124MB
root@ubuntukurs:~# docker run --name postgres_db_new -v /opt/postgres_backup/:/opt/postgres_backup/ -v /var/posgresqldb/:/var/posgresqldb/ -e POSTGRES_PASSWORD=Netology -p 5432:5432 -d bitnami/postgresql:12.10.0
e6fc751d9556985fdc7c53c3126703b66b95600cea5ba84486eafb4ae816ed53
root@ubuntukurs:~# docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS         PORTS                                       NAMES
e6fc751d9556   bitnami/postgresql:12.10.0   "/opt/bitnami/script…"   10 seconds ago   Up 6 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres_db_new

```
проверяем список баз
```
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)

```

восстанавливаем из бэкапа

```
I have no name!@2f779fe7938d:/$ psql -U postgres test_db < /opt/postgres_backup/test_db.backup
```
проверяем базу и список таблиц
```
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(4 rows)

postgres=# \c test_db
You are now connected to database "test_db" as user "postgres".
test_db=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner   
--------+---------+-------+----------
 public | clients | table | postgres
 public | orders  | table | postgres
(2 rows)

```
