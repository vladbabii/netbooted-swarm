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
````
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




