# emqx
Message Queue

EMQX is a cloud-native, MQTT-based, IoT messaging platform designed for high reliability and massive scale. EMQX is a tool in the Message Queue category of a tech stack. 
EMQX is currently the most scalable MQTT broker for IoT applications. It processes millions of MQTT messages in a second with sub-millisecond latency and allows messaging among more than 100 million clients within a single cluster. EMQX is compliant with MQTT 5.0 and 3.x. It’s ideal for distributed IoT networks and can run on the cloud, Microsoft Azure, Amazon Web Services, and Google Cloud. The broker can implement MQTT over TLS/SSL and supports several authentication mechanisms like PSK, JWT, and X.509. Unlike Mosquitto, EMQX supports clustering via CLI, HTTP API, and a Dashboard.

<br>
### Installation of EMQX
```sh
curl -s https://assets.emqx.com/scripts/install-emqx-rpm.sh | sudo bash
yum install epel-release -y ;
yum install -y openssl11 openssl11-devel ;
sudo yum install emqx -y;

# Start and Status Check
sudo systemctl start emqx
systemctl status emqx

#Port Check
ss -ltn 
netstat -tupln | grep emqx

# Reload Daemon and restart emqx after config change
systemctl daemon-reload
systemctl restart emqx

# Admin/Users password reset (not for first-time setup)
emqx ctl admins passwd <Username> <NewPassword>
e.g.: emqx ctl admins passwd admin NewPassword#1234
```

### Check EMQX Web Dashboard
Web Dashboard: http://IP_of_VM_Server:18083/ <br>
`in my case: http://192.168.121.141:18083` <br>
Default-login: <br>
    Default-username: <b>admin</b> <br>
    Default-password: <b>public</b><br>
<br>
![Test Image 1](https://github.com/UnstopableSafar08/emqx/blob/main/Assets/1-login.png) 

```sh
# ------------------------------ Single Node setup  (CentOS7 and RHEL8/9)---------------------------------------------------------------------
official Binary setup : https://www.emqx.io/docs/en/latest/deploy/install-rhel.html
curl -s https://assets.emqx.com/scripts/install-emqx-rpm.sh | sudo bash
sudo yum install emqx -y
sudo systemctl start emqx


# ------------------------------ Install with rpm (recommended Method)------------------------------------------------
wget https://www.emqx.com/en/downloads/broker/5.5.1/emqx-5.5.1-el8-amd64.rpm
sudo yum install emqx-5.5.1-el8-amd64.rpm -y

# ----------------------------- Start the EMQX by systemctl ----------------------------------------
sudo systemctl start emqx
sudo service emqx start
emqx --version
# ----------------------------- Direct Start the EMQX by systemctl ----------------------------------------
emqx start
EMQX 5.5.1 is started successfully!
emqx ctl status
Node 'emqx@127.0.0.1' 5.5.1 is started

# ----------------------------- Service for EMQX ------------------------
[Unit]
Description=EMQ X Broker
After=network.target
[Service]
Type=simple
User=emqx
Group=emqx
ExecStart=/path/to/emqx start
ExecStop=/path/to/emqx stop
Restart=on-failure
[Install]
WantedBy=multi-user.target
# ------------------------------ Main config File ---------------------------------------------------------------------
official Configuration file Link : https://www.emqx.io/docs/en/latest/configuration/configuration.html
Depends on your installation mode, emqx.conf is stored in:
-----------------------------------------------------------------------------
Installation                                  Path
-----------------------------------------------------------------------------
Installed with RPM or DEB package             /etc/emqx/emqx.conf
Running in docker container                   /opt/emqx/etc/emqx.conf
Extracted from portable compressed package    ./etc/emqx.conf
-----------------------------------------------------------------------------
Configuration file path: /etc/emqx
Log file path: /var/log/emqx
Data file path: ``/var/lib/emqx`
# ---- password reset
./bin/emqx ctl admins passwd <Username> <Password>
# ------------------------------ Uninstall -------------------------------
sudo yum remove emqx -y
sudo rm -rf /etc/emqx \
    && sudo rm -rf /var/lib/emqx \
    && sudo rm -rf /var/log/emqx

sudo rm /etc/systemd/system/emqx.service
sudo systemctl daemon-reload
sudo userdel emqx
sudo groupdel emqx


# ------------------------------ Docker container setup ---------------------------------------------------------------------
official Docs : https://www.emqx.io/docs/en/latest/


official Docker Setup link : https://www.emqx.io/docs/en/latest/deploy/install-docker.html
sudo localectl set-locale LANG=C.UTF-8
# Docker Image Pull
docker pull emqx/emqx:5.5.1
# Creating a volume 
mkdir -p /opt/emqx/data /opt/emqx/log
cd /opt/emqx/
# Give a permission to the folder, docker can create a conf files on it
chmod -R 777 /opt/emqx/
# run a docker container of emqx
docker run -d --name emqx \
  -p 1883:1883 -p 8083:8083 \
  -p 8084:8084 -p 8883:8883 \
  -p 18083:18083 \
  -v $PWD/data:/opt/emqx/data \
  -v $PWD/log:/opt/emqx/log \
  emqx/emqx:5.5.1

docker ps -a
# ---- Dashboard Check ----
Web Dashboard : http://IP:18083/
Default-login: 
    username : admin 
    password : public
    new password : emqx#123
# ------------------------------ Cluster creating - Manually ---------------------------------------------------------------------
official Cluster creation Link : https://www.emqx.io/docs/en/latest/deploy/cluster/create-cluster.html#manual-clustering-example
#docker network create for cluster
docker network create emqx-net 

# Docker container 1 : Master node
docker run -d \
    --name emqx1 \
    -e "EMQX_NODE_NAME=emqx@node1.emqx.com" \
    --network emqx-bridge \
    --network-alias node1.emqx.com \
    -p 1883:1883 \
    -p 8083:8083 \
    -p 8084:8084 \
    -p 8883:8883 \
    -p 18083:18083 \
    emqx/emqx:5.5.1
# Docker container 2 : slave node1
docker run -d \
    --name emqx2 \
    -e "EMQX_NODE_NAME=emqx@node2.emqx.com" \
    --network emqx-net \
    --network-alias node2.emqx.com \
    emqx/emqx:5.5.1
# Docker container 2 : join a Master Node, second command is run inside the container  
docker exec -it emqx2 \
    emqx ctl cluster join emqx@node1.emqx.com

# Docker container 3 : slave node2
docker run -d \
    --name emqx3 \
    -e "EMQX_NODE_NAME=emqx@node3.emqx.com" \
    --network emqx-net \
    --network-alias node3.emqx.com \
    emqx/emqx:5.5.1
# Docker container 3 : join a Master Node, second command is run inside the container 
docker exec -it emqx3 \
    emqx ctl cluster join emqx@node1.emqx.com

```
