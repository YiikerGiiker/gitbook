https://book.hacktricks.xyz/network-services-pentesting/137-138-139-pentesting-netbios
https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb

### Initial checks

Scans:
```bash
nmblookup -A <IP>
nbtscan <IP>
enum4linux -a <IP>
```

Determine version:
- [Script](https://github.com/rewardone/OSCPRepo/blob/master/scripts/recon_enum/smbver.sh)

Nmap enumeration:
```bash
nmap --script smb-vuln* -p445 <IP>
```

List shares:
```bash
smbclient --no-pass -L //<IP>
smbmap -H <IP> [-P <PORT>]
crackmapexec smb <IP> -u '' -p '' --shares
```
--> With credentials: `smbclient -L \\\\<SERVER-IP> -U <USERNAME>%<PASSWORD>`
--> With hash: `smbclient -L \\\\<SERVER-IP> -U <USERNAME>%<HASH>`

Connect to shares:
```bash
smbclient --no-pass //<IP>/<Folder>

# Download everything
recurse on
prompt off
mget *
```

Try to connect using guest account:
```bash
nxc smb 192.168.210.159 -u "guest" -p "" --shares
```

Authenticate to share using impacket:
```bash
impacket-smbclient zeus.corp/guest@192.168.210.159 -no-pass -dc-ip 192.168.210.158
```