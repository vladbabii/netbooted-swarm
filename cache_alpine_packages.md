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
You should replace the line
```
set $backend http://mirrors.hostico.ro;
```
with another alpine mirror that's closer to you.

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

Update netboot.ipxe and change the line
```
set mirror http://dl-cdn.alpinelinux.org/alpine
```
to
```
set mirror http://192.168.100.100:4991/alpinelinux
```

# Test settings
On a worker node do 
```
echo "http://192.168.100.100:4991/alpinelinux/v3.18/main" > /etc/apk/repositories 
echo "http://192.168.100.100:4991/alpinelinux/v3.18/community" >> /etc/apk/repositories
apk update
```
then do install any random packages you're normally not uing
```
apk add nginx
apk add mysql
```

# Updating the apkvol
Now we need to update the apkvol because after it gets applied the repositories from it will be used before the q.start script runs.
So when generating the apkvol the script now will be changed from
```
apk del nano
cd /
lbu package thinclient.apkovl.tar.gz 
scp /thinclient.apkovl.tar.gz root@192.168.100.100:/storage/wwww 
apk add nano
```
to
```
apk del nano
echo "http://192.168.100.100:4991/alpinelinux/v3.18/main" > /etc/apk/repositories 
echo "http://192.168.100.100:4991/alpinelinux/v3.18/community" >> /etc/apk/repositories
cd /
lbu package thinclient.apkovl.tar.gz 
scp /thinclient.apkovl.tar.gz root@192.168.100.100:/storage/wwww
echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/main" > /etc/apk/repositories 
echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/community" >> /etc/apk/repositories 
apk add nano
```
This allows you to
* set local package caching server as apk source
* build and copy over apkvol
* set back the default repositories so you can keep working in the apkvol






