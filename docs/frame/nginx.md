# Nginx
## 代理 YRAN 界面
- yarm.conf

```
server {
    listen       10012;
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        proxy_pass http://192.168.0.1:8088;
        subs_filter '主机名' '替换后的IP';
        auth_basic "Please input password";
        auth_basic_user_file /root/nginx-1.8.0/passwd;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```