--создайте новый кластер PostgresSQL 14
sudo pg_createcluster 16 otus --port 7432

--зайдите в созданный кластер под пользователем postgres
sudo -iu postgres
psql -p 7432

--создайте новую базу данных testdb

create database testdb;

--зайдите в созданную базу данных под пользователем postgres

\c testdb
--создайте новую схему testnm
create schema testnm;

--создайте новую таблицу t1 с одной колонкой c1 типа integer
create table t1(c1 int);

--вставьте строку со значением c1=1
insert into t1(c1) select 1;

--создайте новую роль readonly
create role readonly with nologin;

--дайте новой роли право на подключение к базе данных testdb
grant connect on database testdb to readonly;

--дайте новой роли право на использование схемы testnm
grant usage on schema testnm to readonly;

--дайте новой роли право на select для всех таблиц схемы testnm
grant select on all tables in schema testnm to readonly;
alter default privileges in schema testnm grant select on tables to readonly;

--создайте пользователя testread с паролем test123
create user testread with password 'test123';

--дайте роль readonly пользователю testread
grant readonly to testread;

--зайдите под пользователем testread в базу данных testdb

-- предварительно правим pg_hba.conf я просто разрешил всем локаьно подключения
nano /etc/postgresql/16/otus/pg_hba.conf
--local   all             all                                     scram-sha-256

psql -p 7432 -U testread


--сделайте select * from t1;


--получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
ERROR:  permission denied for table t1

--напишите что именно произошло в тексте домашнего задания
--у вас есть идеи почему? ведь права то дали?
--посмотрите на список таблиц
select * from pg_tables

--подсказка в шпаргалке под пунктом 20

-- ответ: не та схема

--а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)

-- ответ: при создании таблицы не указан схема, где создавать

--вернитесь в базу данных testdb под пользователем postgres
--удалите таблицу t1

drop table t1;

--создайте ее заново но уже с явным указанием имени схемы testnm

create table testnm.t1(c1 int);

--вставьте строку со значением c1=1
insert into t1(c1) select 1;

--зайдите под пользователем testread в базу данных testdb
--сделайте select * from testnm.t1;
--получилось?

-- ответ: у меня получилось потому что я при создании роли "перестарался" и выдал alter default privileges, в виду того, 
--        что на работе сейчас часто приходится делать задачи по созданию ролей и выдаче прав

--есть идеи почему? если нет - смотрите шпаргалку
--как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
--сделайте select * from testnm.t1;
--получилось?
--есть идеи почему? если нет - смотрите шпаргалку
--сделайте select * from testnm.t1;
--получилось?
--ура!
--теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
--а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
--есть идеи как убрать эти права? если нет - смотрите шпаргалку

-- у меня 15 версия, прав на создание в паблике нет :)

--если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
--теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
--расскажите что получилось и почему

-- заглядывал в подсказку на первом вопросе про схему, когда создали в паблик, а не в то, на которую права давали
