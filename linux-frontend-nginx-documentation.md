---

## **1️⃣ About Frontend**

* The **frontend service** is the web UI of RoboShop.

* It uses **static content** (HTML, CSS, JS).

* To serve static content → need a **web server**.

* Chosen web server \= **Nginx** (lightweight, high-performance).

---

## **2️⃣ Check Available Nginx Modules**

`dnf module list nginx`

* Shows all versions of **nginx module streams** available in RHEL/CentOS 8/9.

Example output:

 `Name  Stream   Profiles`  
`nginx 1.20     common [d]`  
      `1.24     common`

*   
* `d` means default version.

---

## **3️⃣ Install Specific Version (1.24)**

`dnf module disable nginx -y`

* Disable current default module (maybe 1.20).

`dnf module enable nginx:1.24 -y`

* Enable **Nginx 1.24 stream** (a specific version).

`dnf install nginx -y`

* Installs **nginx** package.

---

## **4️⃣ Enable & Start Service**

`systemctl enable nginx`

* Auto-start nginx on boot.

`systemctl start nginx`

* Start nginx service now.

✅ At this point, visiting server’s IP in browser → shows **default Nginx welcome page**.

---

## **5️⃣ Remove Default Content**

`rm -rf /usr/share/nginx/html/*`

* Default HTML files are stored in `/usr/share/nginx/html`.

* Remove them so we can place RoboShop frontend files.

---

## **6️⃣ Download Frontend Files**

`curl -o /tmp/frontend.zip https://roboshop-artifacts.s3.amazonaws.com/frontend-v3.zip`

* Download RoboShop frontend build as a zip into `/tmp`.

* `-o` \= output file name.

---

## **7️⃣ Extract Content**

`cd /usr/share/nginx/html`  
`unzip /tmp/frontend.zip`

* Move to web root folder.

* Extract frontend.zip content here → files now served by nginx.

✅ Now, browser → RoboShop UI visible.

---

## **8️⃣ Configure Reverse Proxy**

Open Nginx config:

`vim /etc/nginx/nginx.conf`

### **Explanation of Config**

`user nginx;`

* Nginx worker processes run as `nginx` user (security).

`worker_processes auto;`

* Auto-detect CPU cores → optimal worker processes.

`error_log /var/log/nginx/error.log notice;`  
`pid /run/nginx.pid;`

* Error logs go to `/var/log/nginx/error.log`.

* PID of main nginx process stored in `/run/nginx.pid`.

`include /usr/share/nginx/modules/*.conf;`

* Loads extra module configs.

---

### **Events Block**

`events {`  
    `worker_connections 1024;`  
`}`

* Each worker can handle **1024 simultaneous connections**.

---

### **HTTP Block**

`http {`  
    `log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '`  
                      `'$status $body_bytes_sent "$http_referer" '`  
                      `'"$http_user_agent" "$http_x_forwarded_for"';`

    `access_log  /var/log/nginx/access.log  main;`

* Defines **custom log format** (`main`).

* Logs requests to `/var/log/nginx/access.log`.

   `sendfile on;`  
    `tcp_nopush on;`  
    `keepalive_timeout 65;`  
    `types_hash_max_size 4096;`

* Performance tuning:

  * `sendfile` \= efficient file transfers.

  * `tcp_nopush` \= send headers \+ data in one packet.

  * `keepalive_timeout` \= keep connections alive for 65s.

  * `types_hash_max_size` \= MIME types caching size.

   `include /etc/nginx/mime.types;`  
    `default_type application/octet-stream;`

* Maps file extensions (HTML, CSS, JS).

* Fallback → binary stream.

   `include /etc/nginx/conf.d/*.conf;`

* Loads extra configs from `conf.d/`.

---

### **Server Block**

`server {`  
    `listen       80;`  
    `listen       [::]:80;`  
    `server_name  _;`  
    `root         /usr/share/nginx/html;`

* Listens on **port 80** (IPv4 \+ IPv6).

* `server_name _;` \= default server (wildcard).

* Web root → `/usr/share/nginx/html`.

   `include /etc/nginx/default.d/*.conf;`

* Includes additional configs if present.

---

### **Error Pages**

   `error_page 404 /404.html;`  
    `location = /404.html { }`

    `error_page 500 502 503 504 /50x.html;`  
    `location = /50x.html { }`

* Custom error pages for 404 and 5xx errors.

---

### **Image Cache Example**

   `location /images/ {`  
      `expires 5s;`  
      `root   /usr/share/nginx/html;`  
      `try_files $uri /images/placeholder.jpg;`  
    `}`

* Cache images for 5s.

* If image missing → serve `/images/placeholder.jpg`.

---

### **Reverse Proxy for Backend APIs**

   `location /api/catalogue/ { proxy_pass http://localhost:8080/; }`  
    `location /api/user/ { proxy_pass http://localhost:8080/; }`  
    `location /api/cart/ { proxy_pass http://localhost:8080/; }`  
    `location /api/shipping/ { proxy_pass http://localhost:8080/; }`  
    `location /api/payment/ { proxy_pass http://localhost:8080/; }`

* These APIs are **not frontend files** → forward them to backend microservices.

* `proxy_pass http://localhost:8080/;` forwards requests to backend service.

* ⚠️ Must replace `localhost` with **actual backend server IP/hostname**.

---

### **Health Check**

   `location /health {`  
      `stub_status on;`  
      `access_log off;`  
    `}`

* `/health` → shows Nginx status (active connections, requests).

* Logging disabled for this endpoint.

---

## **9️⃣ Restart Nginx**

`systemctl restart nginx`

* Restart service to apply changes.

---

✅ **End result**:

* Static frontend content served by Nginx.

* Backend API calls reverse-proxied to microservices.

* Custom error pages \+ monitoring via `/health`.

