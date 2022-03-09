# rman进行非归档模式的备份和恢复：
案例1：小明在2020年5月30日用rman工具对数据库做了一个非归档模式下的冷备份，在6月2日 建了一些数据库对象在某个用户表空间中，可是由于该用户表空间数据文件丢失，导致数据库无法正常使用，此外当时日志文件已被覆盖，请问这些数据库对象数据还能恢复吗，请用rman工具让恢复数据库正常工作。 
```sql
archive log list;
$ Rman target/
run{
    shutdown immediate;
    startup mount;
    allocate channel c1 type disk;
    allocate channel c2 type disk;
    backup database filesperset 2 format 'd:\backup\1bk\full_%n_%T_%t_%s_%p.bak';
    backup spfile format 'd:\backup\1bk\spfile_%n_%U_%T.bak';
    bakcup archivelog all format 'd:\backup\1bk\arch_%d_%T_%s_%p.bak';
    backup current controlfile format 'd:\backup\1bk\ctl_%d_%T_%s_%p.bak';
    alter database open;
    release channel c1;
    release channel c2;
}
exit

conn /as sysdba
create table test(a number) tablespace users;
insert into test values(1)
commit;
alter system switch logfile;
shutdown immediate;

$ Rman target/
run{
    restore database;
    recover database;
    alter database open resetlogs;
}
exit

conn /as sysdba
select * from temp;
select * from test;
```

案例2：小明在2020年5月30日用rman工具对数据库做了一个归档模式下的数据库全备份，可在6月3日发现所有数据文件和 控制文件都丢失了，导致数据库关闭，无法正常打开，幸好备份的归档日志完好，请问如何使数据库今早打开恢复正常工作，请问丢失的数据文件中的数据能找回吗，请模拟操作
```sql
$ Rman target/  
run{
    allocate channel c1 type disk;
    allocate channel c2 type disk;
    backup database filesperset 2 format 'd:\backup\1bk\full_%n_%T_%t_%s_%p.bak';
    backup spfile format 'd:\backup\hot\spfile_%n_%U_%T.bak';
    sql 'alter system archive log current';
    bakcup archivelog all format 'd:\backup\hot\arch_%d_%T_%s_%p.bak' delete input;
    backup current controlfile format 'd:\backup\hot\ctl_%d_%T_%s_%p.bak';
    release channel c1;
    release channel c2;
}
exit

conn /as sysdba
create table data00(id number,name varchar2(30)) tablespace data02;
insert into data00 values(1, 'data01');
commit
shutdown immediate
startup

$ Rman target/
restore controlfile from 'd:\backup\hot\CTL_ORCL_20200614_60_1.BAK';
alter database mount;
run{
    allocate channel c1 type disk;
    allocate channel c2 type disk;
    restore database;
    recover database;
    alter database open resetlogs;
}
exit

conn /as sysdba
```

案例3：小明在2020年5月30日用rman工具对数据库做了一个归档模式下的数据库全备份，在6月4日建了一个非常重要的表，6月6日发现这个重要表在的数据文件所在的磁盘损坏，导致数据库无法正常打开工作，且重要数据丢失，而工作人员无法在短时间内对磁盘进行维修，请问你能帮他恢复数据库正常工作并找回重要表的提交数据。模拟操作
```sql
$ Rman target/
run{
    allocate channel c1 type disk;
    allocate channel c2 type disk;
    backup database filesperset 1 format 'd:\backup\hot\full_%n_%T_%t_%s_%p.bak';
    backup spfile format 'd:\backup\hot\spfile_%n_%U_%T.bak';
    sql 'alter system archive log current';
    bakcup archivelog all format 'd:\backup\hot\arch_%d_%T_%s_%p.bak' delete input;
    backup current controlfile format 'd:\backup\hot\ctl_%d_%T_%s_%p.bak';
    release channel c1;
    release channel c2;    
}
exit

create table tb(id number, name varchar2(30)) tablespace data02;
insert into tb values(1, 'a');
insert into tb values(2, 'b');
commit;
shutdown immediate
startup

$ Rman target/
run{
    allocate channel c1 type disk;
    allocate channel c2 type disk;
    restore database;
    recover database;
    alter database open resetlogs;
    sql 'alter system archive log current';
}
exite

alter database open;
select * from td;
```