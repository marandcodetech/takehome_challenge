# Solution for problem no.3:


# Disk Full Diagnosis and Recovery (Nginx VM)

## Diagnosis

To diagnose the issue, follow these steps.

### 1. Confirm the Problem Is Disk Space (Not Inode Exhaustion)

Run the following commands:

```bash
df -h
```

Checks disk space usage.

```bash
df -i
```

Checks inode usage.

Interpretation:

- If `df -h` is high → the problem is most likely **large files**.
- If `df -i` is high → the problem is **too many small files**.

---

### 2. Identify Which Folder Consumes the Most Space

Run:

```bash
sudo du -xhd1 / | sort -h
```

Shows folder sizes from the root filesystem.

For Nginx VMs specifically check `/var`:

```bash
sudo du -xhd1 /var | sort -h
```

---

# Probable Causes

## 1. Log Files Exploded (Most Common)

Check Nginx logs:

```bash
ls -lh /var/log/nginx/
```

Large `access.log` or `error.log` files indicate this is the cause.

### Recovery

Compress existing logs:

```bash
sudo gzip /var/log/nginx/access.log
```

Then truncate the file:

```bash
sudo truncate -s 0 /var/log/nginx/access.log
```

**Recommended long-term solution**

Send logs to centralized logging systems:

- CloudWatch
- ElasticSearch / Logstash / Kibana (ELK stack)

---

## 2. Core Dumps From Crashes

Usually caused by repeated Nginx crashes.

Check for core dumps:

```bash
ls -lh /var/lib/systemd/coredump/
```

If files of **2GB or larger** exist, this may be the cause.

### Impact

- Indicates service crashes
- Possible upstream or OS instability
- Potential memory corruption or configuration issues

### Recovery

- Remove core dump files
- Disable or limit core dump generation
- Investigate root cause of the crashes

---

## 3. Log Floods From 502 / 504 Errors

If upstream services fail, NGINX logs every error request.

Possible causes:

- Upstream cluster down
- Health checks failing
- Misconfigured DNS
- Certificate mismatch

### Recovery

- Fix upstream services
- Reduce error logging
- Rate-limit abusive traffic
- Add WAF protection if bots are involved

---

## 4. Attack Scenarios

Malicious traffic may generate massive logs.

Examples:

- Bot attacks
- DDoS traffic
- Random IPs hitting nonexistent endpoints

This causes disk exhaustion because:

- Every request is logged
- Massive numbers of `404` responses

### Recovery

- Enable rate limiting
- Use Cloudflare or another WAF
- Disable logging for static 404 errors
- Block suspicious IP ranges

---

# Immediate Emergency Rescue

If disk is completely full:

1. Stop Nginx

```bash
sudo systemctl stop nginx
```

2. Delete the largest logs

3. Restart Nginx

```bash
sudo systemctl start nginx
```

---

# Most Common Scenario: Log Explosion

Check **logrotate configuration**:

```bash
cat /etc/logrotate.d/nginx
```

This file defines automatic log rotation.

Check excessive logging level:

```bash
grep error_log /etc/nginx/nginx.conf
```

If it shows:

```
debug
```

This is problematic in production because **all debugging activity gets logged**, rapidly increasing disk usage.

---

# Cyberattack Scenario Diagnosis

Check the latest log entries:

```bash
sudo tail -100 /var/log/nginx/access.log
```

Look for:

- Thousands of requests from the same IP
- Random endpoint scans

Example patterns:

```
/wp-admin
/phpmyadmin
/admin
/cgi-bin
```

- Extremely high request rates

---

### Identify Top Requesting IPs

Run:

```bash
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head
```

This shows the **top IP addresses generating traffic**.

If one IP appears **hundreds of thousands or millions of times**, it is likely the attacker.

---

# Cyberattack Recovery

### 1. Truncate Logs to Recover Disk Space

```bash
sudo truncate -s 0 /var/log/nginx/access.log
```

---

### 2. Block Attacker IPs

Using iptables:

```bash
sudo iptables -A INPUT -s <ip> -j DROP
```

Using UFW:

```bash
sudo ufw deny from <ip>
```

---

### 3. Enable Rate Limiting in Nginx

```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
```

Then apply:

```nginx
limit_req zone=one burst=20 nodelay;
```

---

### 4. Add Upstream Protection

Place a protection layer in front of the Nginx VM, for example:

- Cloudflare WAF
- CDN-based rate limiting
- DDoS protection services
