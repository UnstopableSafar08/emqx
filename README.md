# emqx
Message Queue

EMQX is a cloud-native, MQTT-based, IoT messaging platform designed for high reliability and massive scale. EMQX is a tool in the Message Queue category of a tech stack. 
EMQX is currently the most scalable MQTT broker for IoT applications. It processes millions of MQTT messages in a second with sub-millisecond latency and allows messaging among more than 100 million clients within a single cluster. EMQX is compliant with MQTT 5.0 and 3.x. It’s ideal for distributed IoT networks and can run on the cloud, Microsoft Azure, Amazon Web Services, and Google Cloud. The broker can implement MQTT over TLS/SSL and supports several authentication mechanisms like PSK, JWT, and X.509. Unlike Mosquitto, EMQX supports clustering via CLI, HTTP API, and a Dashboard.
****
## Architecture
![emqx-arch](https://github.com/UnstopableSafar08/emqx/blob/main/Assets/architecture_image.f5sZc1A2.png)

## Test Cluster overview
![testArch](https://github.com/UnstopableSafar08/emqx/blob/main/Assets/local-cluster-overview.png)

## EMQX - Installation
```sh 
curl -s https://assets.emqx.com/scripts/install-emqx-rpm.sh | sudo bash
yum install epel-release -y ;
yum install -y openssl11 openssl11-devel ;
sudo yum install emqx -y;

# Start and Status Check
sudo systemctl start emqx
systemctl status emqx

#Port Check (18083, 8083, 1883, 8081, 8883, 8084, 4370, 5370)
ss -ltn 
netstat -tupln | grep emqx

# Reload Daemon and restart emqx after config change
systemctl daemon-reload
systemctl restart emqx

# Admin/Users password reset (not for first-time setup)
emqx ctl admins passwd <Username> <NewPassword>
e.g.: emqx ctl admins passwd admin NewPassword#1234
```
***
## EMQX - Login Dashboard
Web Dashboard: http://IP:18083/ <br>
`sh In my Case  : http://192.168.121.141:18083` <br>
Default-login: <br>
    Default-username: admin <br>
    Default-password: public <br>
### Login page
![login-dashboard](https://github.com/UnstopableSafar08/emqx/blob/main/Assets/1-login.png)
***
