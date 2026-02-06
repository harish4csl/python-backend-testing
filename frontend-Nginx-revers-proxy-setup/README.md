# 🌐 Frontend EC2 Setup with Nginx Reverse Proxy (AWS)

This README documents the **final, clean, working setup** of a **Frontend EC2** running **Nginx** as a **reverse proxy**, serving a modern HTML/JS UI and forwarding API calls to the backend via `/api`.

This document is written **after successful debugging** and reflects the **exact configuration that works**.

---

## 🧱 Architecture Overview

```
User Browser
    |
    |  HTTPS / HTTP (80 / 443)
    v
AWS Application Load Balancer (ALB)
    |
    |  Forward traffic
    v
Frontend EC2 (Nginx)
    |
    |-- Serves Static UI (HTML / CSS / JS)
    |-- Reverse Proxies /api → Backend
    v
Backend EC2 / Service (API)
```

### Key Concept

* **HTML & JavaScript run in the browser**
* **Nginx runs on Frontend EC2**
* **JavaScript calls `/api`**
* **Nginx forwards `/api` to backend**

---

## 📁 Directory Structure (Final)

```
Frontend EC2
├── /usr/share/nginx/html/
│   └── index.html          # Frontend UI (served to browser)
│
├── /etc/nginx/
│   ├── nginx.conf
│   └── conf.d/
│       └── frontend.conf   # Reverse proxy config
│
└── Nginx Service (running)
```

---

## ⚙️ Step-by-Step Setup Guide

### 1️⃣ Launch Frontend EC2

* Amazon Linux 2 / Amazon Linux 2023
* Open ports in **Security Group**:

  * `80` from ALB SG (preferred)
  * OR `80` from `0.0.0.0/0` (for testing)

---

### 2️⃣ Install Nginx

```bash
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

Verify:

```bash
systemctl status nginx
```

---

### 3️⃣ Remove Apache (If Installed)

> ❗ Apache (`httpd`) must NOT conflict with Nginx

```bash
sudo systemctl stop httpd
sudo yum remove httpd -y
```

---

### 4️⃣ Place Frontend UI Code

📍 **Location**:

```
/usr/share/nginx/html/index.html
```

```bash
sudo vi /usr/share/nginx/html/index.html
```

📌 **IMPORTANT**

* Only **one** `index.html`
* File must have **read permissions**

```bash
sudo chmod 644 /usr/share/nginx/html/index.html
```

---

### 5️⃣ Frontend UI Code (WORKING – DO NOT MODIFY)

> ✅ This exact code is verified and working after debugging

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>User Management Portal | Mr.Harish</title>
  ...
  <script>
    const backendBase = "/api"; // Backend routed via Nginx reverse proxy
    ...
  </script>
</head>
<body>
</body>
</html>
```

📌 **Critical Line**

```js
const backendBase = "/api";
```

This ensures:

* No backend IP exposed
* Works behind ALB
* Browser → Frontend → Backend

---

### 6️⃣ Create Nginx Reverse Proxy Config

📍 **File Location**:

```
/etc/nginx/conf.d/frontend.conf
```

```bash
sudo vi /etc/nginx/conf.d/frontend.conf
```

### ✅ Working Reverse Proxy Configuration

```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    # Reverse proxy for backend API
    location /api/ {
        proxy_pass http://BACKEND_PRIVATE_IP:PORT/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

🔁 Replace:

* `BACKEND_PRIVATE_IP`
* `PORT` (e.g. 5000, 8080)

---

### 7️⃣ Validate & Restart Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

### 8️⃣ Attach Frontend EC2 to ALB Target Group

* Target Type: **Instance**
* Protocol: **HTTP**
* Port: **80**

Health Check:

* Path: `/`
* Success Code: `200`

---

## 🔐 Security Group Rules (Testing Phase)

| Component    | Port     | Source      |
| ------------ | -------- | ----------- |
| ALB          | 80       | 0.0.0.0/0   |
| Frontend EC2 | 80       | ALB SG      |
| Backend EC2  | API Port | Frontend SG |

> ⚠️ `0.0.0.0/0` is OK **only for testing**

---

## 🧠 How Traffic Flows (Packet-Level)

1. User opens ALB DNS
2. ALB forwards request to Frontend EC2
3. Nginx serves `index.html`
4. JS runs **in browser**
5. JS calls `/api/users`
6. Nginx forwards `/api` → Backend
7. Backend returns JSON
8. Browser renders data

---

## ✅ Verification Checklist

✔ ALB DNS opens UI
✔ Page loads CSS & JS
✔ `/api/users` works
✔ No blank page
✔ No mixed content
✔ No backend IP in browser

---

## 🚨 Common Mistakes (Avoid These)

❌ Calling backend via private IP in JS
❌ Running Apache + Nginx together
❌ Putting proxy config inside `index.html`
❌ Missing trailing slash in `proxy_pass`
❌ Wrong file permissions

---

## 🎯 Final Result

You now have:

* Clean Frontend EC2
* Nginx reverse proxy
* ALB-compatible setup
* Secure backend access
* Production-style architecture

---

## 📌 Author

**Harish**
Multicloud | DevOps | AWS Training

---

## ⭐ Recommendation

If this lab helped you:

* Fork it
* Star it ⭐
* Use it for real-world demos

Happy Learning 🚀
