(This is a work in progress and more features like cron based text alarms are coming)  
# lab_iotstack
Almost everything you need to replace LabView: NodeRed, InfluxDB, and Grafana, plus Portainer to manage it from a browser.  

The concept is you have a server on a network with your instruments running modbus or any other communication protocal supported. Anywhere you have SSH access to the server a client can view and manage the services using ssh-port-forwarding. The only external port and traffic should be through SSH so security risks are very low. The server can be almost anything, and any OS as far as I know: Win/Mac/Linux/RPi4. I recommend however a real server running Ubuntu. 

# Setup System
## Secure your server (if you need to)
Things to consider are creating a docker user, removing root ssh access, closing any ports, setting up a firewall, add IP ban.
Here is a good guide https://www.linode.com/docs/guides/set-up-and-secure/


## Install Docker 
To get started install docker (w/ docker compose) on your server following the [official Docker instructions](https://docs.docker.com/desktop/install/ubuntu/).  

Clone this repository onto the server. I recommend doing this on a drive where you want to store the data, by default it will create a directory in this repository called `volumes` where the images will save their persistant data: flows, database tables, settings, etc.  

(Optional) Create a ssh key for automatic passwordless autnentication from your client computers by running `ssh-keygen -t rsa -b 4096`. Upload the public key via ssh:  `scp ~/.ssh/id_rsa.pub user@server-ip:~/.ssh/my_client.pub`. Add the key to your `authorized_keys` file by logging in and running the following command: `cat ~/.ssh/my_client.pub >> ~/.ssh/authorized_keys`.
To automatically login and forward all ports to your client machine create a `config` file in ~/.ssh on the client with the following:
```
Host XYZ
  HostName server-ip
  IdentityFile ~/.ssh/id_rsa
  User username
  LocalForward 9000 localhost:9000
  LocalForward 1880 localhost:1880
  LocalForward 3000 localhost:3000
  LocalForward 8086 localhost:8086
```
where `XYZ` is the alias for the ssh command, set it to `lab` or whatever you want to remember. `server-ip` is the ssh address, typically a IPV4 or domain-name if you have one assigned, `username` is your ssh user. The ports are:  
9000 - [Portainer](https://www.portainer.io/): manages the docker stack  
1880 - [NodeRed](https://nodered.org/): Event based programmer similar to LabView  
3000 - [Grafana](https://grafana.com/): Displays data in databases in realtimeish  
8086 - [InfluxDB](https://www.influxdata.com/): Timeseries database NodeRed will write to and Grafana will display.  
You can add other ports if you want to also run a Jupyter-Notebook, VNC, etc, on the server.  
Login to the server from the client with `ssh XYZ`.

# Customizing
There are a few things you may want to customize before building the stack:
1) If you would prefer the volumes where persistant data is stored, now is the time to do that
2) Change the passwords to InfluxDB if your local network is not private (or do it anyway). These passwords are used to connect the various services running on the local network only to the database only and should (probably) not be accessable through your firewall. You can always connect to the database remotly though SSH port forwarding. The password setting is in the docker-compose file.
4) Add any additional nodes/flows you may want to the `services/nodered/Dockerfile`. You can see what's available here (https://flows.nodered.org/). By default InfluxDB, Modbus, PID, and Telnet are included.

## User account
Make a new user account for 

# Running
To run the docker stack use the following command inside the repository:
`docker compose up -d`

# Setup Containers
Nodes are now hopefully running. They can connect between containers with the hostnames: influxdb, nodered, grafana.
Login to each container:
## Portainer (localhost:9000)
First login should prompt for creation of admin user and password. Then prompt to what system it's managing, connect to the local docker instance.
## InfluxDB (localhost:8086)
The initial user and password was inituser/initpass unless you changed it above. 
## NodeRed (localhost:1880)
For whatever reason this does not have users or paswords
## Grafana
The default user/pass is admin/admin. When you login it will prompt to change the password, and you can create new users later with varring privlages. 

# Using Containers
A short demonstration showing the use of the docker stack should be enought to get started.  
[![DEMO](https://img.youtube.com/vi/OSR40Dc_1BY/0.jpg)](https://www.youtube.com/watch?v=OSR40Dc_1BY)
