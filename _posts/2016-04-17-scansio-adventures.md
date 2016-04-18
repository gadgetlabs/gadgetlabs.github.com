---
layout: post
title: "scans.io adventures"
description: ""
category: 
tags: []
---
{% include JB/setup %}

# Download the files from scans.io

```bash
wget https://scans.io/zsearch/dmw8z7wkktwcs0jg-143-imap-starttls-full_ipv4-20160410T022854-zgrab-results.json.lz4 
sudo apt-get install liblz4-tool
sudo pip install python-geoip
sudo pip install python-geoip-geolite2
```

Process the file - on a free instance on AWS the extracted size is too big, so process on stdout

```bash
lz4 -dc dmw8z7wkktwcs0jg-143-imap-starttls-full_ipv4-20160410T022854-zgrab-results.json.lz4 | ./process.py > uk.json
```

contents of process.py

```python
#!/usr/bin/env python
import sys
import json
from geoip import geolite2

for line in sys.stdin:
	line = line.strip()
	s = json.loads(line)
	
	match = geolite2.lookup(s['ip'])
	if match is None:
		continue

	if match.country is 'GB':
		print line
```

Next narrow the results to BT wholesale - e.g. businesses

```bash
$ whois -h whois.radb.net -- '-i origin AS2856' | grep -Eo "([0-9.]+){4}/[0-9]+" | head
192.151.16.0/20
61.28.211.0/24
61.28.219.0/24
205.157.82.0/24
193.102.37.0/24
193.35.182.0/23
193.35.184.0/21
193.35.192.0/22
193.35.196.0/23
193.218.114.0/24


$ whois -h whois.radb.net -- '-i origin AS2856' | grep -Eo "([0-9.]+){4}/[0-9]+" > ranges.txt
```



```python
#!/usr/bin/env python
import sys
import json
from netaddr import *

bt = IPSet()
with open("ranges.txt", "r") as fp:
	for line in fp.readlines():
		ip = line.strip()
		bt.add(IPNetwork(ip))

for line in sys.stdin:
	line = line.strip()
	s = json.loads(line)
	if s['ip'] in bt:
		print line
```

