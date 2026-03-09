# Solution for problem no.3:

**Diagnosis:**
To diagnose, there are several steps that can be performed:
Confirm it’s actually disk problem (not inode exhaustion):
Run the following commands:
$ df -h; this will check space usage.
$ df -i; this will check inode usage.
		If df -h is high, the problem is most likely caused by large files.
		If df -i is high, there are too many small files.

Find which folder consumes the most space, run these commands:
sudo du -xhd1 / | sort -h
This will show all folders sizes in a sorted way.
sudo du -xhd1 /var | sort -h
This will show all /var folders sizes in a sorted way which is specific for Nginx VM.

**Probable Causes:**
Log files exploded (most common):
Check using the following command:
ls -lh /var/log/nginx/
We may see several logs files with large sizes. Then we may determine this as the probable cause.

**Recovery:**
Compress the necessary log files first using this command:
sudo gzip /var/log/nginx/access.log
Then followed by truncating it:
sudo truncate -s 0 /var/log/nginx/access.log
It is much better to send logs to CloudWatch or ElasticSearch stacks (ElasticSearch, Logstash, Kibana/ELK)

**Core dumps from crashes**
This is mostly caused by nginx repeatedly crashing.
Check using the following command:
ls -lh /var/lib/systemd/coredump/
If there are several files with size 2GB or more, this is the probable cause. The impacts:
I. Indicates crashes
Ii. Possible upstream or OS instability
Iii. Potential memory corruption or misconfigurations
		
**Recovery:**
Remove core dumps.
Disable or limit core dumps.
Investigate why it crashed.

**Log floods from 502/504 errors**
If upstream services are failing, NGINX logs every 502/504.

**Possible causes:**
Upstream cluster down
Health check failing
Misconfigured DNS
Certificate mismatch

**Recovery:**
Fix upstream
Reduce error logging
Rate-limit abusive traffic
Add WAF rules if bots involved
Attack scenarios
Consider malicious traffic:
Bots attacks
DDoS generating huge logs
Random IPs hitting nonexistent endpoints
Disk explodes because:
Every request gets logged
404 storm
Recovery:
Enable rate limiting
Use Cloudflare / WAF
Disable logging for static 404s
Block bad IP ranges

**Immediate Emergency Rescue:**
Stop nginx service immediately
Delete largest logs
Restart nginx


**The most common scenarios:**
Log files exploded (most common)

**Recovery:**
Check logrotate configurations:
cat /etc/logrotate.d/nginx
This will show the logs rotations configurations.
Check excessive log level:
grep error_log /etc/nginx/nginx.conf
If it shows something “debug”,  then this is problematic in production. All debugging activities get logged which causes the logs to explode.

**Cyberattacks scenarios**
**Mitigations:**
Check using this command:
sudo tail -100 /var/log/nginx/access.log

**Things to look for:**
Simultaneous thousands of requests from the same IP.
Random URL scanning:
/wp-admin, /phpmyadmin, /admin, /cgi-bin
Extremely high request rate

**Check frequency:**
sudo awk '{print $1}' /var/nginx/log/access.log | sort | uniq -c | sort -nr | head
This command shows top IPs. If one IP was found to hit up to a million times, then this is the primary suspect.

**Recovery:**
Truncate logs to regain disk size: 
sudo truncate -s 0 /var/log/nginx/access.log
Identify attacker IPs and block temporarily:
sudo iptables -A INPUT -s <ip> -j DROP
Can be done using ufw command:
sudo ufw deny from <ip>
Enable rate limiting in nginx:
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
Then followed by:
limit_req zone=one burst=20 nodelay;
Put protection upstream, for example: put Cloudflare WAF in front of this Nginx VM.

