# 思源笔记 Docker部署



[官方部署文档](https://github.com/siyuan-note/siyuan/blob/master/README_zh_CN.md#docker-%E9%83%A8%E7%BD%B2)



自用Docker-Compose



```yaml
version: "3.9"
services:
  main:
    container_name: siyuan
    image: b3log/siyuan
    command: ['--workspace=/siyuan/workspace/', '--accessAuthCode=CHANGE_YOUR_AUTH_CODE']
    ports:
      - 6806:6806
    volumes:
      - /path/for/local/dir/siyuan/workspace:/siyuan/workspace
    restart: unless-stopped
    environment:
      # A list of time zone identifiers can be found at https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
      - TZ=CN
```



Nginx 部署



Nginx 代理思源笔记服务，注意需要添加 `location /ws { ... }` 配置以使用 WebSocket



```nginx
 server {
        listen 80;
        server_name  a.b.c;
        client_max_body_size 10m;
        location / {
            proxy_pass http://siyuan;
            proxy_set_header HOST $host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /ws {
            proxy_pass http://siyuan;
            proxy_read_timeout 60s;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'Upgrade';
        }
     }
```



如果打不开思源笔记,查看F12,显示 websocket问题则是配置 /ws 出现了问题



如果访问思源网页返回了json提示:

`访问鉴权失败，请刷新或者重新打开`

可以尝试添加

```nginx
            proxy_set_header If-Modified-Since "";
            proxy_set_header If-None-Match "";
```



以下自用nginx

```nginx

server {
    listen 443 ssl;

    http2 on;
    
    server_name https://siyuan.url; 

    include /config/nginx/ssl.conf;

    client_max_body_size 10m;

#自定替换url
    location / {
                
            proxy_pass https://siyuan.url;
            proxy_redirect http:// https://;
    }

    location /ws {
            proxy_pass https://siyuan.url;
            proxy_read_timeout 60s;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'Upgrade';
    #添加下面2个
            proxy_set_header If-Modified-Since "";
            proxy_set_header If-None-Match "";
    }
}

```

