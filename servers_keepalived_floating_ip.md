# Keepalived for multiple servers
If you have multiple servers that offer the same files and use them as multiple docker managers, you need a floating ip to have a single entry point for booting the workers

Servers with their ips:
* 1 - .201
* 2 - .202
* 3 - .203

Floating ips for services:
* fttpd server: .210:69
* http boot server: .211:64080
* alpine cache server: .212:4991
* docker registry server: .2013:5000



# WIF

## Install
```
mkdir -p /storage/server_keepalived
apt update && apt install -y keepalived
```

## Configure
touch /storage/server_keepalived/check_alpine_caching_repository.sh
touch /storage/server_keepalived/check_docker_caching_registry.sh
touch /storage/server_keepalived/check_tftpd.sh
touch /storage/server_keepalived/check_boothttp.sh
chmod +x /storage/server_keepalived/*.sh
```

edit /storage/server_keepalived/check_alpine_caching_repository.sh
```
#!/bin/bash
STATUS=$(netstat -antepul | grep 4991 | grep -i listen | wc -l)
if [ "$STATUS" = "1" ]; then
  # working
  exit 0
else
  # not working
  exit 1
fi
```

edit /storage/server_keepalived/check_docker_caching_registry.sh
```
#!/bin/bash
STATUS=$(netstat -antepul | grep 5000 | grep -i listen | wc -l)
if [ "$STATUS" = "1" ]; then
  # working
  exit 0
else
  # not working
  exit 1
fi
```

edit /storage/server_keepalived/check_tftpd.sh
```
#!/bin/bash
STATUS=$(netstat -antepul | grep 69 | grep -i listen | wc -l)
if [ "$STATUS" = "1" ]; then
  # working
  exit 0
else
  # not working
  exit 1
fi
```

edit /storage/server_keepalived/check_boothttp.sh
```
#!/bin/bash
STATUS=$(netstat -antepul | grep 64080 | grep -i listen | wc -l)
if [ "$STATUS" = "1" ]; then
  # working
  exit 0
else
  # not working
  exit 1
fi
```
