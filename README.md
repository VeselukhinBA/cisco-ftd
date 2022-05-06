# Cisco_FTD

## Ссылка

* FMC task stuck - [link to original doc](https://dependencyhell.net/2021/fmc-task-stuck-deleting-broken-tasks-from-database)

## FMC task stuck
Периодически после внесения измений конфигурации на FTD при выполнении Deploy происходит зависание процесса. Рестрат не помогает.
Нужно удалить зависшую задачу из БД. Ниже описан порядок действий для software releases >=6.3.0.

**Step 01**: Switch to bash (expert) shell and change to root user
```shell
> expert
 admin@fmc01:~$ sudo su - 
```

**Step 02**: Execute OmniQuery.pl to search for running tasks
```shell
root@fmc01:~#  OmniQuery.pl -db mdb -e "select status,category,hex(uuid) from notification where status=7;"
+--------+-------------------+----------------------------------+
| status | category          | hex(uuid)                        |
| 7      | task:category.150 | bb0bba970b4c4423927b8f7d237edd0b |
+--------+-------------------+----------------------------------+
1 rows in set 
```

**Step 03**: Copy the uuid of your task and delete it from Sybase using OmniQuery.pl:
```shell
root@fmc01:~#  OmniQuery.pl -db mdb -e 'delete from notification where uuid=unhex("bb0bba970b4c4423927b8f7d237edd0b");'
Query OK, 1 rows affected (0.000 sec)  
```

**Step 04**: Double-check to make sure the entry was successfully deleted from notification table
```shell
OmniQuery.pl -db mdb -e "select status,category,hex(uuid) from notification where status=7;"

Empty set (0.001 sec) 
```

**Step 05**: Restart FMC Processes for changes to take effect
```shell
root@fmc01:~#  /etc/rc.d/init.d/console restart
```

After your FMC has successfully restarted, login to the UI and make sure the task / deployment is no longer listed
