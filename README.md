(This is a work in progress and more features like cron based text alarms are coming)  
# lab_iotstack
Almost everything you need to replace LabView: NodeRed, InfluxDB, and Grafana, plus Portainer to manage it from a browser.  

The concept is you have a server on a network with your instruments running modbus or any other communication protocal supported. Anywhere you have SSH access to the server a client can view and manage the services using ssh-port-forwarding. The only external port and traffic should be through SSH so security risks are very low. The server can be almost anything, and any OS as far as I know: Win/Mac/Linux/RPi4. I recommend however a real server running Ubuntu. 

# Setup
To get started install docker engine on your server following the [official Docker instructions](https://docs.docker.com/desktop/install/ubuntu/).  

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
1) If you would prefer the volumes where persistant data is stored, now is the best time to do that
2) Change the passwords to InfluxDB if your local network is not private (or do it anyway). These passwords are used to connect the various services to the database only and should (probably) not be accessable through your firewall.
3) Add any additional nodes/flows you may want to the `services/nodered/Dockerfile`. You can see what's available here (https://flows.nodered.org/). By default InfluxDB, Modbus, PID, and Telnet are included.

# Running
To run the docker stack use the following command inside the repository:
`docker compose up -d`

