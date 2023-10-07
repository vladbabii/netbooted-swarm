# NetBooted Swarm

# May thanks to
netboot server 
* https://www.youtube.com/watch?v=xmVGIdScCQA
* https://www.apalrd.net/posts/2022/alpine_pxe/#download--configure-ipxe
* 
netboot apkvol based on
* https://www.apalrd.net/posts/2022/alpine_vdiclient/
* https://www.youtube.com/watch?v=r-TnP06K-gE


# Install Ubuntu Server LTS minimal install
... there are plenty of guides on this. Just make sure to set a static ip

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

# Install and configure netboot server components
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


# Setup netboot os files

We'll use latest alpine, 3.18 at the point of this post
```
cd /storage/www
wget https://dl-cdn.alpinelinux.org/alpine/v3.18/releases/x86_64/alpine-netboot-3.18.4-x86_64.tar.gz
tar -xzf alpine*
rm alpine-netboot*
chown -R www-data:www-data /storage/www
```

# Build ipxe image for x86 / x64
```
apt install -y make gcc binutils perl mtools mkisofs syslinux liblzma-dev isolinux
cd /srv
git clone https://github.com/ipxe/ipxe.git
cd ipxe/src
```

edit netboot.ipxe
```
#!ipxe

#Init networking
dhcp

#Networking info we got from the DHCP server
echo next-server is ${next-server}
echo filaneme is ${filename}
echo MAC address is ${net0/mac}
echo IP address is ${ip}

#Set flavor to lts
set flavor lts
echo flavor is ${flavor}

#Set command line 
set cmdline modules=loop,squashfs quiet
echo cmdline is ${cmdline}

#Server address
set server http://${next-server}
echo server is ${server}

#Kernel file
set vmlinuz ${server}/boot/vmlinuz-${flavor}
echo vmlinuz is ${vmlinuz}
set initramfs ${server}/boot/initramfs-${flavor}
echo initramfs is ${initramfs}

#Modloop file
set modloop ${server}/boot/modloop-${flavor}
echo modloop is ${modloop}

#Repository for apk
#Update this if you'd like a newer version of Alpine
#Alternatively, set branch to edge for the absolutel latest
set mirror http://dl-cdn.alpinelinux.org/alpine
set branch v3.15
set repo ${mirror}/${branch}/main
echo repo is ${repo}

#apkovl file - set this if you want to apply
#an apkovl file to configure the Alpne instance
set apkovl ${server}/thinclient.apkovl.tar.gz
echo apkovl is ${apkovl}

#Uncomment this if you want to see the information before continuing
#prompt Press any key to continue

#Kernel, initrd
#For EFI, we need to tell the kernel the initrd filename. For BIOS it doens't hurt to leave the initrd argument.
#If you want to use Alpine bare, use this line:
#kernel ${vmlinuz} ${cmdline} alpine_repo=${repo} modloop=${modloop} initrd=initramfs-${flavor}
#If you want to use Alpine with an apkovl, use this line:
kernel ${vmlinuz} ${cmdline} modloop=${modloop} apkovl=${apkovl} initrd=initramfs-${flavor}
initrd ${initramfs}

#Boot
boot

#Pause if errors
prompt Some error occurred, press any key to continue
```

edit build.sh
```
#Build BIOS version (x86 but should boot into x64 environment)
make bin-i386-pcbios/undionly.kpxe EMBED=netboot.ipxe
cp bin-i386-pcbios/undionly.kpxe /storage/tftp/

#Build EFI version (x86)
#make bin-x86_64-efi/ipxe.efi EMBED=netboot.ipxe
#cp bin-x86_64-efi/ipxe.efi /storage/tftp/ipxe64.efi

#The APKOVL we are using is for x64, so not building ipxe32.efi right now
#Also not building arm variants for this project
```
I only needed the BIOS version but if you need EFI then also uncomment the make.. and cp.. lines under Build EFI

# Configure DHCP server
... here youre on your own, plenty of guides on how to do it for all routers. Set the server ip to the one you used on the ubuntu server above




