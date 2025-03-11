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
  scan_ssid=1
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
