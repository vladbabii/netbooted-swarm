In q.sh add after hostname setup (replace 192.168.168.100 with your rsyslog server)
```
## rsyslog installed?
if test -f "/etc/rsyslog.conf"; then
  echo "rsyslog installed"
else
  echo "installing rsyslog"
  apk add rsyslog
  echo "*.* @192.168.168.100" >> /etc/rsyslog.conf
  sleep 1
fi

## rsyslog running ?
pgrep -x /usr/sbin/rsyslogd >/dev/null && echo "rsyslog running already" || service rsyslog start
```
If using docker then the /etc/docker/daemon.json must contain
```
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "udp://192.168.168:100:514"
  }
}
```
also you if your server is different than the main one you need to add in q.sh when hostname is checked
```  echo "<ip>    <hostname of rsyslog server>"        >> /etc/hosts
```
