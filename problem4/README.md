# Solution problem no.4:


# Deployment Review and Fixes

## Problem Identification

### 1. API Port Mismatch

Node API starts on:

```js
app.listen(3000)
```

But NGINX forwards traffic to port **3001**:

```nginx
location /api/ {
    proxy_pass http://api:3001;
}
```

**Impact**

NGINX attempts to connect to port `3001`, but the Node API runs on `3000`.

**Result**

- Intermittent **502 Bad Gateway**
- API appears unreachable


---

### 2. PostgreSQL Connection Pool Misconfiguration

PostgreSQL is forced to:

```sql
ALTER SYSTEM SET max_connections = 20;
```

Meanwhile the Node service uses:

```js
const pool = new Pool()
```

Default pool settings:

- max connections = **10**

If the API scales or multiple requests arrive simultaneously:

- Multiple requests open DB connections simultaneously
- PostgreSQL quickly exhausts the **20 connection limit**

**Impact**

```
remaining connection slots are reserved for superuser connections
```

This causes random API failures.


---

### 3. No Health Checks in docker-compose.yaml

The compose file uses:

```yaml
depends_on:
```

However `depends_on` **does not wait for service readiness**, only container start.

Therefore:

- API may start before **PostgreSQL** or **Redis**
- Startup crashes may occur
- Services fail to connect


---

### 4. No API Restart Process

If the API crashes due to a DB connection failure, the container **remains down**.


---

# Diagnosis Process

## 1. Check Container Logs

Run:

```bash
docker compose logs nginx
docker compose logs api
docker compose logs postgres
```

Expected result:

```
connect() failed (111: Connection refused) while connecting to upstream
```

This indicates a **port mismatch**.


---

## 2. Verify Network Connectivity

Enter the NGINX container:

```bash
docker compose exec nginx sh
```

Test API connectivity:

```bash
curl http://api:3001
```

Fails.

Test again:

```bash
curl http://api:3000
```

Works.

Root cause confirmed.


---

## 3. Inspect Running Ports

Inside the API container:

```bash
netstat -tlnp
```

Output:

```
LISTEN 0.0.0.0:3000
```

Confirms port mismatch.


---

# Fixes Applied

## 1. Fix Nginx Port Configuration

Updated configuration:

```nginx
server {
    listen 80;

    location = / {
        return 200 "Welcome to the platform\n";
    }

    location /api/ {
        proxy_pass http://api:3000;
    }
}
```


---

## 2. Modify PostgreSQL Connection Limits

Better configuration:

```
max_connections = 100
```

Mount initialization scripts:

```yaml
postgres:
  image: postgres:15
  environment:
    POSTGRES_PASSWORD: postgres
  volumes:
    - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
```


---

## 3. Configure Node Connection Pool

```js
const pool = new Pool({
  host: process.env.DB_HOST,
  user: "postgres",
  password: "postgres",
  database: "postgres",
  port: 5432,
  max: 5,
  idleTimeoutMillis: 30000,
});
```


---

## 4. Add Health Checks

### API Healthcheck

```yaml
api:
  build: ./api
  environment:
    - DB_HOST=postgres
    - REDIS_HOST=redis
  depends_on:
    - postgres
    - redis
  healthcheck:
    test: ["CMD", "wget", "-qO-", "http://localhost:3000/status"]
    interval: 10s
    timeout: 3s
    retries: 5
```

### PostgreSQL Healthcheck

```yaml
postgres:
  image: postgres:15
  environment:
    POSTGRES_PASSWORD: postgres
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 10s
    retries: 5
```


---

## 5. Add Restart Policy

```yaml
restart: unless-stopped
```

This ensures services recover automatically.


---

# Monitoring and Alerts

The observability stack should include:

**Metrics**

- Prometheus
- Grafana

**Logs**

- Centralized logging via ELK stack


---

# Production Prevention Strategy

## CI/CD Safeguards

Add pipeline checks for:

- Port mismatches
- Health endpoints
- Container readiness


---

## Infrastructure Testing

Automated integration tests:

```bash
docker compose up
```

Then run API tests.


---

## Environment Configuration

Avoid hardcoded ports.

```js
const PORT = process.env.PORT || 3000;
app.listen(PORT);
```


---

## Add Connection Pooling Layer

Example:

- **pgBouncer**


---

## Additional Nginx Settings

```nginx
proxy_connect_timeout 5s;
proxy_read_timeout 30s;
proxy_next_upstream error timeout http_502 http_503;
```


---

# Final Architecture

```
Client
   ↓
Nginx :80
   ↓
Node API :3000
   ↓
PostgreSQL + Redis
```

With all these fixes implemented, the system becomes **stable, scalable, and observable**.
