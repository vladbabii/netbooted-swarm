# Local caching of alpine packages
This removes the requirement of fresh nodes to download all the packages (docker,openssh, etc) from the internet

On the server
```
mkdir -p /storage/alpine_caching_repository/main
chmod -R 777 /storage/alpine_caching_repository/main
```

edit /storage/alpine_caching_repository/main.nginx.conf
```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    gzip  on;

    proxy_force_ranges on;
    proxy_cache_path /cache levels=1:2 keys_zone=my_cache:10m max_size=10000
                    inactive=12m use_temp_path=off;

    server {
        location / {
            resolver 127.0.0.11 valid=10s;
            set $backend http://mirrors.hostico.ro;
            proxy_pass $backend;
            proxy_cache my_cache;
            proxy_read_timeout 120s;
            proxy_cache_valid 200 301 302 12m;

            add_header 'Access-Control-Allow-Origin' '*';
        }
    }
}
```


In portainer create a new stack with (replace <serverhostname> with the actual hostname)
```
version: '3.2'

services:
  apk_main:
    image: nginx:1-alpine
    container_name: apk_main
    ports:
      - "4991:80"
    volumes:
      - /storage/alpine_caching_repository/main:/cache
      - /storage/alpine_caching_repository/main.nginx.conf:/etc/nginx/nginx.conf:ro
    restart: always
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.hostname == <serverhostname>
```
