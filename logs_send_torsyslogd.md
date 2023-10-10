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
