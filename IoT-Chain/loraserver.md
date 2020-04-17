****************************************
# LoRa Servers
****************************************
ports : 22, 80, 8080, 1700

*******************************************************
==============
## LoRa Gateway Bridge
===============
 
===========
### Prerequisites

Linux Ubuntu 18.04 LTS.

Mosquitto

===========

### Mosquitto Installation
==============
sudo apt update
sudo apt install mosquitto mosquitto-clients

 ===========

*****The LoRa Server project provides pre-compiled binaries packaged as Debian (.deb) packages.In order to activate this repository, execute the following commands:

sudo apt install apt-transport-https dirmngr
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
sudo echo "deb https://artifacts.loraserver.io/packages/2.x/deb stable main" | sudo tee /etc/apt/sources.list.d/loraserver.list
sudo apt-get update 

 ===========

******Install the LoRa Gateway Bridge.

sudo apt-get install lora-gateway-bridge 


********Start LoRa Gateway Bridge.

sudo systemctl start lora-gateway-bridge 

systemctl status lora-gateway-bridge 

******Start lora-gateway-bridge on boot.

sudo systemctl enable lora-gateway-bridge 


*******************************************************
==============
## LoRa Server
===============


****Install PostgreSQL.

sudo apt-get install postgresql 

*****Install Redis server.

sudo apt-get install redis-server 

****LoRa Server needs its own database. To create a new database, start the PostgreSQL prompt as the postgres user.

sudo -u postgres psql 

*****Within the PostgreSQL prompt, enter the following queries (Note: The database password is: 'loradbpassword'):

create role loraserver_ns with login password 'loradbpassword'; 


create database loraserver_ns with owner loraserver_ns;

\q 

***** Verify if the user and database have been setup correctly, try to connect to it:

psql -h localhost -U loraserver_ns -W loraserver_ns 
\q 

*****Install LoRa Server.

sudo apt-get install loraserver 


***loraserver configuration file is installed at: /etc/loraserver/loraserver.toml
**** loraserver executable is installed at: /usr/bin/loraserver

****Modify the LoRa Server configuration file /etc/loraserver/loraserver.toml.
 
sudo su 
cd /etc/loraserver 
nano loraserver.toml 

*****Make the following changes (if needed):


dsn="postgres://loraserver_ns:loradbpassword @localhost/loraserver_ns?sslmode=disable" 
automigrate=true
name="EU_863_870"
timezone="Local"
server="tcp://127.0.0.1:1883"



******Start LoRa Server.

sudo systemctl start loraserver 

sudo systemctl status loraserver 

**** Start LoRa Server on boot.

sudo systemctl enable loraserver 


==============
## LoRa App Server
==============
 

****LoRa App Server persists the gateway data into a PostgreSQL database.
LoRa App Server needs its own database. To create a new database, start the PostgreSQL prompt as the postgres user:

sudo -u postgres psql 

****Within the PostgreSQL prompt, enter the following queries: 

create role loraserver_as with login password 'loradbpassword'; 
create database loraserver_as with owner loraserver_as; 

****Enable the pg_trgm (trigram) extension.
 
\c loraserver_as
 create extension pg_trgm;
 \q 

****Verify if the user and database have been setup correctly, try to connect to it: 
psql -h localhost -U loraserver_as -W loraserver_as 
 \q 

****Install LoRa App Server.
sudo apt-get install lora-app-server 

****lora-app-server configuration file is installed at: /etc/lora-app-server/lora-app-server.toml
****lora-app-server executable is installed at: /usr/bin/lora-app-server



****Create a JSON Web Token (jwt). Open a terminal.

openssl rand -base64 32

****Result:   0FTIjnMh+0bWgl5d4csdZ5yIdHnrVQAg0lOgFg/+lSw=



****Modify the lora-app-server configuration file. /etc/lora-app-server/lora-app-server.toml.
 
sudo su 
cd /etc/lora-app-server
nano lora-app-server.toml 

****Make the following changes (if needed):

dsn="postgres://loraserver_as:loradbpassword@ localhost/loraserver_as?sslmode=disable"
jwt_secret="e3+eD7zcVFJF3EFpPnM1oMj02DqUZxt5wR4IfPBpbtA="
server="tcp://localhost:1883"
public_host="localhost:8001" 

****Explication 
Note 1: dsn
Given you used the password dbpassword when creating the PostgreSQL database.
Note 2: jwt_secret
jwt_secret, see step 6.
Note 3: server
MQTT broker address and port
Note 4: public_host
The Internal API Server is used by LoRa Server to communicate with LoRa App Server 

****start
sudo systemctl start lora-app-server

 sudo systemctl enable lora-app-server 

============
## LoRa Gateway Bridge configuration
==========
  
**** Modify LoRa Gateway Bridge configuration.Modify the LoRa Gateway Bridge configuration file: 
/etc/lora-gateway-bridge/lora-gateway-bridge.toml.

sudo su 
cd /etc/lora-gateway-bridge
nano lora-gateway-bridge.toml 

****Make the following changes (if needed):
server="tcp://127.0.0.1:1883" 

Note: server
MQTT broker address and port 

Type: exit (After changes)


***********check it
****Check if all required services are running
 systemctl status ttn-gateway
 systemctl status mosquitto 
 systemctl status lora-gateway-bridge
 systemctl status loraserver
 systemctl status lora-app-server


============
## Packet Forwarder configuration 
==========
  







