### 📁 Structure

```bash
nginx-502-fix/
├── broken-nginx.conf
├── fixed-nginx.conf
├── app.py
├── service.service
├── README.md
```

---

## 🔥 1. `broken-nginx.conf`

This intentionally causes 502.

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:9999;  # WRONG PORT (causes 502)
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 🔥 2. `fixed-nginx.conf`

Correct version.

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:8000;  # CORRECT PORT
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 🧠 3. `app.py` (simple FastAPI app)

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "App is running"}
```

Run with:

```bash
uvicorn app:app --host 127.0.0.1 --port 8000
```

---

## ⚙️ 4. `service.service` (optional but strong proof)

```ini
[Unit]
Description=FastAPI App
After=network.target

[Service]
User=www-data
WorkingDirectory=/path/to/app
ExecStart=/usr/bin/uvicorn app:app --host 127.0.0.1 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 📄 5. `README.md` (THIS is what sells you)

Don’t mess this up. Copy this clean:

````md
# Nginx 502 Bad Gateway Fix

## Problem
The application was returning a **502 Bad Gateway** error when accessed through nginx.

## Cause
Nginx was configured to proxy requests to an incorrect upstream port:
- nginx → 127.0.0.1:9999 ❌
- actual app → 127.0.0.1:8000 ✔️

Since no service was running on port 9999, nginx returned a 502 error.

## Fix
Updated nginx configuration to point to the correct upstream service:

```nginx
proxy_pass http://127.0.0.1:8000;
````

Restarted nginx after applying the fix.

## Result

* Application became accessible
* 502 error resolved
* Stable connection between nginx and backend

## Key Learnings

* Always verify upstream service ports
* Check if backend service is running
* Validate nginx configuration before reload

