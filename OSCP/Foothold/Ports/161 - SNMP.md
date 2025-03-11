https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-snmp/index.html

Discover using Nmap:
```bash
sudo nmap -sU --open -p 161 <IP>
```


### Brute force community string

```bash
nmap -sU --script snmp-brute -p 161 <IP>

hydra -P /opt/SecLists/Discovery/SNMP/common-snmp-community-strings.txt <IP> snmp

onesixtyone -c /opt/SecLists/Discovery/SNMP/common-snmp-community-strings-onesixtyone.txt <IP> 2>&1   
```


### Snmpwalk

Enumeration using snmpwalk: (public = default community string)
```bash
snmpwalk -c public -v1 -t 10 <IP>
```

Enumerate users:
```bash
snmpwalk -c public -v1 <IP> 1.3.6.1.4.1.77.1.2.25
```

Enumerate running processes:
```bash
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.4.2.1.2
```

Enumerate installed software:
```bash
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.25.6.3.1.2
```

Enumerate TCP listening ports:
```bash
snmpwalk -c public -v1 <IP> 1.3.6.1.2.1.6.13.1.3
```


### Detailed

snmpcheck:
```bash
snmpcheck 192.168.138.42
```

snmpbulkwalk:
```bash
sudo apt-get install snmp-mibs-downloader
sudo download-mibs
# Finally comment the line saying "mibs :" in /etc/snmp/snmp.conf
sudo vi /etc/snmp/snmp.conf

snmpbulkwalk -c public -v2c <IP> .
```
