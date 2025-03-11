
Import module:
```bash
Import-Module .\PowerView.ps1
```

Enumeration commands:
```bash
Get-NetDomain

# Users
Get-NetUser
Get-NetUser | select cn
Get-NetUser | select cn,pwdlastset,lastlogon

# Groups
Get-NetGroup | select cn
Get-NetGroup "Sales Department" | select member

# Computers
Get-NetComputer
Get-NetComputer | select operatingsystem,dnshostname
Get-NetComputer | select dnshostname,operatingsystem,operatingsystemversion

# Admin on another system?
Find-LocalAdminAccess

# Active sessions
Get-NetSession -ComputerName files04 -Verbose

# Get ACL's
Get-Acl -Path HKLM:SYSTEM\CurrentControlSet\Services\LanmanServer\DefaultSecurity\ | fl
Get-ObjectAcl -Identity stephanie
Get-ObjectAcl -Identity "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights

# Convert SIDs to names:
"S-1-5-21-1987370270-658905905-1781884369-512","S-1-5-21-1987370270-658905905-1781884369-1104","S-1-5-32-548","S-1-5-18","S-1-5-21-1987370270-658905905-1781884369-519" | Convert-SidToName

# SPN's
Get-NetUser -SPN | select samaccountname,serviceprincipalname

# Domain shares
Find-DomainShare

# GPP decrypt
gpp-decrypt "+bsY0V3d4/KgX3VJdO/vyepPfAN1zMFTiQDApgR92JE"
```
