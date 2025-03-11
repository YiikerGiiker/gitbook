
**Don't forget $ at the end of a user**
- e.g. `svc_apache$`

Spray the credentials against each server & service:
```bash
# SMB
nxc smb <IP> -u <user> -p <pass>
nxc smb <IP> -u <user> -p <pass> --shares
nxc smb <IP> -u <user> -p <pass> --users

# Other
nxc ftp <IP> -u <user> -p <pass>
nxc wmi <IP> -u <user> -p <pass>
nxc ldap <IP> -u <user> -p <pass>
nxc mssql <IP> -u <user> -p <pass>
nxc ssh <IP> -u <user> -p <pass>
nxc rdp <IP> -u <user> -p <pass>
nxc winrm <IP> -u <user> -p <pass>
```
>[!info]
>- **Try everything with --local-auth!**
>-  **Try evil-winrm regardless of the output (may fail)**
>- **Also possible with hashes using -H**

Bloodhound usage:
```bash
# From Kali
bloodhound-python -c all -d <domain> -u <user> -p <pass> --zip -ns <dc-ip>

# On Windows
powershell -ep bypass
Import-Module .\Sharphound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory <directory>

# Start bloodhound
sudo neo4j start
bloodhound
```
>[!info]
>- **Clear BloodHound DB, restart to avoid issues**
>- **Bloodhound fails? --> dnschef: [link](https://arz101.medium.com/vulnlab-trusted-c7c26ff00740)**


SPN? (svc_mssql)
**SILVER TICKET ATTACK**
[[3_Attacks]]
