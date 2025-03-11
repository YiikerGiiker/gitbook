https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap


Try to find users & possible passwords stored in the description field:
```bash
nmap -n -sV --script "ldap* and not brute" <IP>

ldapsearch -x -H ldap://<IP> -D '' -w '' -b "DC=<1_SUBDOMAIN>,DC=<TLD>"
```

Check --> [[1_Services]]