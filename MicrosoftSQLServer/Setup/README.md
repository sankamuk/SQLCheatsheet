# Setup Microsoft SQL Server

## Linux Docker Version

> Note you should have a ***Linux host with Docker installed*** to start the playbook.

- Run conatiner with SQL Server

```
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Admin@123" \
   -p 1433:1433 --name sql1 --hostname sql1 \
   -d mcr.microsoft.com/mssql/server:2019-latest
```

> Note the password for SA user is being set to Admin@123, feel free to change.

- Check container is running

```
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE                                        COMMAND                  CREATED      STATUS       PORTS                    NAMES
f7a4d36f267a   mcr.microsoft.com/mssql/server:2019-latest   "/opt/mssql/bin/perm…"   7 days ago   Up 7 hours   0.0.0.0:1433->1433/tcp   sql1
```

- To login to conatiner

```
docker exec -it sql1 "bash"
```

- Inside container you can connect to your DB using sqlcmd

```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "Admin@123"
```

### Restore a Database in your conatiner instance

> Lets suppose you have a North Wind Database backup, i.e. northwind.bak

- Copy it inside the container

```
docker cp northwind.bak sql1:/tmp/
```

- Run sqlcmd inside the container to list out logical file names and paths inside the backup.

```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "Admin@123" -Q "RESTORE FILELISTONLY FROM DISK = N'/tmp/northwind.bak'"
```

Output would be like below:

```
LogicalName                                                                                                                      PhysicalName
                                                                                                                                                        Type FileGroupName
                                                Size                 MaxSize              FileId               CreateLSN                   DropLSN                     UniqueId                             ReadOnlyLSN                 ReadWriteLSN                BackupSizeInBytes    SourceBlockSize FileGroupId LogGroupGUID                         DifferentialBaseLSN         DifferentialBaseGUID                 IsReadOnly IsPresent TDEThumbprint                              SnapshotUrl

-------------------------------------------------------------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---- -------------------------------------------------------------------------------------------------------------------------------- -------------------- -------------------- -------------------- --------------------------- --------------------------- ------------------------------------ --------------------------- --------------------------- -------------------- --------------- ----------- ------------------------------------ --------------------------- ------------------------------------ ---------- --------- ------------------------------------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Northwind                                                                                                                        D:\MSSQLDataFiles\UNTC\DB_All\DB\NORTHWND.MDF D    PRIMARY3538944       35184372080640                    1                           0                           0 00000000-0000-0000-0000-000000000000                           0 0              3407872             512           1 NULL                                           41000000022000037 A4223A2B-1587-4154-9A23-0C40EE0DD2B7          0         1 NULL                                       NULL

Northwind_log                                                                                                                    D:\MSSQLDataFiles\UNTC\DB_All\DB\NORTHWND_log.ldf L    NULL 3932160       35184372080640                    2                           0                           0 1ACEC54C-E29C-4A81-BA2D-79A02F730824                           0 0                    0             512           0 NULL                                                           0 00000000-0000-0000-0000-000000000000          0         1 NULL                                       NULL


(2 rows affected)
```

- Restore Database

```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "Admin@123" -Q 'RESTORE DATABASE NORTHWND FROM DISK = "/tmp/db_backup/northwind.bak" WITH MOVE "Northwind" TO "/var/opt/mssql/data/NORTHWND.MDF", MOVE "Northwind_log" TO "/var/opt/mssql/data/NORTHWND_log.ldf"'
```

- Verify

```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "Admin@123" -Q 'SELECT Name FROM sys.Databases'
```


