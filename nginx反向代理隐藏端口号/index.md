# nginx反向代理隐藏端口号


```nginx
server {
    listen       80;
    server_name  **************;
    auth_basic off;
    location / {
        proxy_pass    http://************:8081;
        proxy_set_header Host $host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 60;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
    }
}

```

