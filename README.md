# NetBooted Swarm

# Install Ubuntu Server LTS minimal install
...

# After first boot install some useful tools
```
apt remove -y --autoremove snapd
apt update
apt upgrade -y
apt install -y \
  dialog \
  screen \
  htop \
  nano \
  dnsutils \
  iputils-ping \
  net-tools \
  iperf \
  iputils-ping \
  cron
systemctl enable cron
```

# Let's install and configure netboot server components
```
apt-get install tftpd-hpa
mkdir -p /storage/tftp
chmod -R 777 /storage/tftp/
touch /storage/tftp/undionly.kpxe
```
edit /etc/default/tftpd-hpa

from
```
  TFTP_DIRECTORY="/srv/tftp"
```
to
```
  TFTP_DIRECTORY="/storage/tftp"
```

then restart tftp server
```
/etc/init.d/tftpd-hpa restart
```
next we'll configure a http server to keep our files
```
apt install nginx -y
mkdir /storage/www
chown -R www-data:www-data /storage/www
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

edit /etc/nginx/nginx.conf
```
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    sendfile on;
    access_log /var/log/nginx/access.log;
    keepalive_timeout 3000;
    server {
        listen 80;
        root /storage/www;
        index index.html index.htm;
        server_name localhost;
        client_max_body_size 32m;
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /var/lib/nginx/html;
        }
    }
}
```
and restart ngingx
```
systemctl restart nginx
```


