# emqx
Message Queue

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
