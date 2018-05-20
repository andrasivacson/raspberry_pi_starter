
get a rapbian imagefrom here:
https://www.raspberrypi.org/downloads/raspbian/
install it on an sdcard using etcher: https://etcher.io/

## Setting up your WIFI credentials:

1. using this guide: https://learn.adafruit.com/adafruits-raspberry-pi-lesson-3-network-setup/setting-up-wifi-with-occidentalis
tldr: edit the following file: sudo nano /etc/network/interfaces
auto lo
```
iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
auto wlan0


iface wlan0 inet dhcp
        wpa-ssid "ssid"
        wpa-psk "password"
```
2. alternatively using sudo raspi-config, turn on SSH as well!!!

## Setting up static IP:

https://www.raspberrypi.org/learning/networking-lessons/rpi-static-ip-address/

1. TLDR: sudo nano /etc/dhcpcd.conf
```
interface eth0

static ip_address=192.168.1.23/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1

interface wlan0

static ip_address=192.168.1.23/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```
2. sudo reboot
3. set host in windows to point to your pi: 
https://support.rackspace.com/how-to/modify-your-hosts-file/
TLDR: add in C:\Windows\System32\drivers\etc\hosts
```
192.168.1.23 raspberrypi
```
4. windows bash: ssh pi@192.168.1.23 or raspberrypi

5. optionally you can add hosts here: 
/etc/hosts
/etc/hostname


## Set up samba server(for transfering files to rpi easily):
https://www.raspberrypi.org/magpi/samba-file-server/
TLDR: 
 
1. udo apt-get install samba samba-common-bin

2. sudo mkdir -m 1777 /share

3. configure samba: 	
sudo leafpad /etc/samba/smb.conf
```
[share]
Comment = Pi shared folder
Path = /share
Browseable = yes
Writeable = Yes
only guest = no
create mask = 0777
directory mask = 0777
Public = yes
Guest ok = yes
```
4. restart server:

sudo /etc/init.d/samba restart

5. browse the shared folder from you PC \\RASPBERRYPI log in if guest mode is turned off 

6. try connecting to a PC share with : smbclient //PCADDRESS/folder -U username

## Set up login to use ssh keys
http://www.rebol.com/docs/ssh-auto-login.html
TLDR:
On server:

cd .ssh
```
ssh-keygen -t rsa  (hit return through prompts)
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
rm id_rsa.pub
```
On client:
```
cd .ssh
scp raspberrypi:.ssh/id_rsa pi.rsa
chmod 600 pi.rsa
echo "Host pi" >> config
echo "User pi" >> config
echo "Hostname raspberrypi" >> config
echo "IdentityFile ~/.ssh/pi.rsa" >> config 
```

## Follow the guide to install docker: 
https://blog.alexellis.io/getting-started-with-docker-on-raspberry-pi/

TLDR: 
1. curl -sSL https://get.docker.com | sh

2. Set Docker to auto-start
```
$ sudo systemctl enable docker
```
You can now reboot the Pi, or start the Docker daemon with: 
```
$ sudo systemctl start docker
```

3. Enable Docker client
The Docker client can only be used by root or members of the docker group.

4. Add pi or your equivalent user to the docker group:
```
$ sudo usermod -aG docker pi
```
## installing postgres container

```
docker pull arm32v7/postgres
docker run --name pi-postgres -e POSTGRES_PASSWORD=mysecretpassword -d arm32v7/postgres
```

Some useful commands:
https://medium.com/statuscode/dockercheatsheet-9730ce03630d
```
docker version
docker images
docker ls

docker search
docker pull
docker build
```
## creating a swarm
https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/
```
docker info
docker swarm init
docker swarm join
docker swarm join-token worker
docker node ls

```

docker build -t dummypage .
docker run --name dummycontainer -p 8080:80 dummypage

docker rmi dummypage --force
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)