# Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

## 1. Mục tiêu

Cấu hình Nginx làm Reverse Proxy cho ứng dụng Spring Boot.

* Nginx lắng nghe trên port 80.
* Trang tĩnh được phục vụ từ `/var/www/html/`.
* Các request có đường dẫn `/api/` được chuyển tiếp đến Spring Boot tại `127.0.0.1:8082`.

## 2. Các file cấu hình

### `index.html`

File được đặt tại:

```text
/var/www/html/index.html
```

File này hiển thị thông tin sinh viên và mã lớp.

### `spring-proxy.conf`

File cấu hình Nginx:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8082/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 3. Kiểm tra cấu hình Nginx

Thực hiện:

```bash
sudo nginx -t
```

Kết quả:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

## 4. Kiểm tra trang tĩnh

Thực hiện:

```bash
curl -I http://localhost/
```

Kết quả:

```text
HTTP/1.1 200 OK
Server: nginx/1.28.3 (Ubuntu)
Content-Type: text/html
```

Trang tĩnh được Nginx phục vụ thành công.

## 5. Kiểm tra ứng dụng Spring Boot

Ứng dụng Spring Boot chạy tại:

```text
127.0.0.1:8082
```

Kiểm tra trực tiếp:

```bash
curl -i http://127.0.0.1:8082/health
```

Kết quả:

```text
HTTP/1.1 200
Content-Type: text/plain;charset=UTF-8

Spring Boot is running on port 8082
```

## 6. Kiểm tra Reverse Proxy

Thực hiện:

```bash
curl -i http://localhost/api/health
```

Kết quả:

```text
HTTP/1.1 200
Server: nginx/1.28.3 (Ubuntu)
Content-Type: text/plain;charset=UTF-8

Spring Boot is running on port 8082
```

Điều này chứng minh request `/api/health` đã được Nginx chuyển tiếp thành công đến ứng dụng Spring Boot.