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

```

```