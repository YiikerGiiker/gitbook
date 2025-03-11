### Basic enum

Identify IP address of host:
```
host www.megacorpone.com
```
- Uses A record by default

Using different record type: (txt also possible)
```
host -t mx megacorpone.com
```
- Lowest priority number used to forward mail


### Manual Enum

DNS brute-forcing to find subdomains:
```bash
for ip in $(cat list.txt); do host $ip.megacorpone.com; done
```
- List.txt contains a list of subdomains
- **SecLists project** (good wordlist)

Use reverse lookups to request the hostname of an IP range:
```bash
for ip in $(seq 200 254); do host 51.222.169.$ip; done | grep -v "not found"
```
- Since we found a range of IP's used we can further enumerate the range using reverse lookups to find more subdomains


### Automatic Enum

Automatic DNS enumeration:
- dnsrecon: 
	- `dnsrecon -d <domain> -t std`
		- Standard enumeration
	- `dnsrecon -d <domain> -D <wordlist> -t brt`
		- Bruteforce subdomains using dictionary
- dnsenum: 
	- `dnsenum <domain>`


### Living off the land

Living off the land:
- nslookup `nslookup <domain>`
	- Resolve A record for domain
- `nslookup -type=TXT <domain> <dns-server-ip>
	- Use the DNS server IP to resolve TXT records related to the domain
