# Create your own VPN server with WireGuard in Docker

## Prerequisites
+ Linux Server running Ubuntu 20.04 LTS or newer
  
*For installing Docker on other Linux distriubtions or different versions than Ubuntu 20.04 LTS, follow the [official installation instructions](https://docs.docker.com/install/).*

## Install Docker, and Docker-Compose
You can still install Docker on a Linux Server that is not running Ubuntu, however, this may require different commands!

### Install Docker
```
sudo apt update && sudo apt upgrade
```
```
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
```
sudo apt update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
### Check if Docker is installed correctly
```
sudo docker run hello-world
```
## Install Docker-Compose
Download the latest version
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```
sudo chmod +x /usr/local/bin/docker-compose
```
### (Optional) Add your linux user to the docker group
```
sudo usermod -aG docker $USER
```
```
newgrp docker
```
### Check if Docker-Compose is installed correctly
```
sudo docker-compose --version
```
## Set up Wireguard in Docker
### Create a new Docker-Compose file
Create a new folder in the /opt directory.
```
sudo mkdir /opt/wireguard-server
```
```
sudo chown user:user /opt/wireguard-server
```
```
cd /opt/wireguard-server
```

You can also use your personal home folder /home/<your-username>, this may require different permissions.

Create a new file docker-compose.yml file, please refer to the [linuxserver/wireguard](https://hub.docker.com/r/linuxserver/wireguard) documentation.
```
vi docker-compose.yaml
```
Replace the `<your-server-url>` with the public IP address of your WireGuard Server, because your clients will need to connect from outside your local network. You can also set this to auto, the docker container will automatically determine your public IP address and use this in the client's configuration.
```
version: "2.1"
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - SERVERURL=<your-server-url> #optional
      - SERVERPORT=51820 #optional
      - PEERS=1 #optional
      - PEERDNS=auto #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
    volumes:
      - /opt/wireguard-server/config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```
### Start the WireGuard Server
```
docker-compose up -d
```
## Connect your laptop / desktop
install wireguard 
+ [Windows](https://download.wireguard.com/windows-client/wireguard-installer.exe)
+ [Mac OS](https://itunes.apple.com/us/app/wireguard/id1451685025?ls=1&mt=12)
+ Ubuntu
  ```
  sudo apt install wireguard resolvconf
  ```
## Copy peer.conf file
+ Windows (`<br>`)
  You can copy the file using [WinSCP](https://winscp.net/download/WinSCP-6.1.1-Setup.exe)
  
+ Linux / MacOS
  

Distribute the config files to clients
You could also use the docker image for your clients. But I think it's more practical for a client to install WireGuard directly on the host OS. If you want to know how to do that, you can also refer to my article about WireGuard installation and configuration on Linux.
```
docker exec -it wireguard wg
```
When you have started the WireGuard container, it should automatically create all configuration files in your **./config folder**. All you need to do is to copy the corresponding **./config/peer1/peer1.conf** file to your client and use that as your **wg0.conf**, for instance. If you want to connect mobile phones you can also just scan the **peer1.png QR code**, to print the QR code to the console, simply use the following command:

```
docker exec -it wireguard /app/show-peer <peer-number>
```
### (Optional) Add additional clients
If you want to add additional clients, you simply can increase the PEERS parameter in the docker-compose.yaml file. After changing this value you need to restart your 
docker container with the ` --force-recreate ` parameter.
```
docker-compose up -d --force-recreate
```
