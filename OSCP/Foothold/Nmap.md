
### TCP

Initial scan all ports
```bash
 nmap -vv -p- <IP> -oN nmap/initial
```
--> Add `-Pn` if ICMP requests aren't allowed (eg Windows)

Targeted scan:
```bash
nmap -sC -sV -p<port1>,<port2> -oN nmap/detailed
```

Vulnerability scanning:
```bash
sudo nmap --script vuln <IP> -oN nmap/vuln
```
>[!Example]
>`sudo nmap --script=smb-vuln* <IP> -oN nmap/smb-vuln`


### UDP

UDP scan
```bash
sudo nmap -vv -sU <IP> -oN nmap/udp
```
--> Add `-Pn` if ICMP requests aren't allowed (eg Windows)
--> Add `-T4` if scan is too slow
