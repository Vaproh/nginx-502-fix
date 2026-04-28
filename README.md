# Nginx 502 Bad Gateway – Debug & Fix

## Problem
While accessing the application through nginx, the server was returning a **502 Bad Gateway** error.

The backend application was expected to be running, but nginx was unable to successfully proxy requests.

---

## Investigation

To identify the issue, I went through a basic debugging flow:

- Checked nginx status to ensure it was running  
- Verified backend service (FastAPI) was active  
- Inspected nginx configuration for upstream settings  
- Reviewed logs for connection errors  

### Commands used:
```bash
systemctl status nginx
ps aux | grep uvicorn
journalctl -u nginx
````

---

## Root Cause

The issue was caused by an incorrect upstream configuration in nginx.

* nginx was pointing to: `127.0.0.1:9999` ❌
* backend app was running on: `127.0.0.1:8000` ✔️

Since no service was running on port `9999`, nginx failed to connect and returned a **502 error**.

---

## Fix

Updated the nginx configuration to use the correct upstream:

```nginx
proxy_pass http://127.0.0.1:8000;
```

Then reloaded nginx:

```bash
sudo systemctl restart nginx
```

---

## Result

* Application became accessible through nginx
* 502 Bad Gateway error resolved
* Stable connection between nginx and backend restored

---

## Key Takeaways

* Always verify the backend service is running and accessible
* Double-check upstream ports in nginx configuration
* Logs (`journalctl`, nginx error logs) are critical for debugging
* Small misconfigurations can break entire deployments

---

## Notes

This is a common production issue when:

* services are moved to different ports
* configs are copied without verification
* backend fails silently

Understanding the debugging process is more important than just applying the fix.
