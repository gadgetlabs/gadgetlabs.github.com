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
