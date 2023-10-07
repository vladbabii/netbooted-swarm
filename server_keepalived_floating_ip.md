# Keepalived for multiple servers

Install
```
mkdir -p /storage/server_keepalived
apt update && apt install -y keepalived
touch /storage/server_keepalived/check_alpine_caching_repository.sh
touch /storage/server_keepalived/check_docker_caching_registry.sh
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

edit check_docker_caching_registry.sh
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
