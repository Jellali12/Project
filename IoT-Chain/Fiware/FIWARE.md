****************************************
##  <p align="center"> FIWARE LoRaWAN IoT Agent </p>
****************************************


## Introduction
<p style='text-align: justify;'> 
This installation and administration guide covers the FIWARE LoRaWAN IoT Agent service. For details on the installation of LoRaWAN IoT Agent itself please refer to FIWARE IOT AGENT FOR LORAWAN PROTOCOL. </p>


## Getting Started
<p style='text-align: justify;'> These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. </p>


## Contents 


 [Architecture](#architecture)
[Prerequisites](#prerequisites)
[Node.js](#node-js)  
[Mosquitto](#mosquitto)
[Docker](#docker)
[Mongo](#mongo)
[Context Brocker](#context-brocker)
[IoT Agent Installation](#iot-agent-installation)

## Architecture
(image)



## Prerequisites
===========
Linux Ubuntu 18.04 LTS.
Node.js
Mosquitto
Docker
Mongodb
Context Broker

===========

### Node.js Installation

- Enable the NodeSource repository and install necessary packages (with sudo privileges):
```ruby
sudo curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash 
```
- Install Node.js and npm by typing:
```ruby 
sudo apt install nodejs 
 ```
- Verify versions:
```ruby 
node --version 
npm --version
 ```





### Mosquitto Installation
```ruby 
sudo apt update
sudo apt install mosquitto mosquitto-clients
```


### Docker Installation

- Install docker:
```ruby 
sudo apt install docker.io
```
- Start and Automate Docker (to set Docker service to run at startup):
```ruby 
sudo systemctl start docker
sudo systemctl enable docker
```
 
### Mongodb Installation 

- to install mongodb with docker:
```ruby 
sudo docker run --name mongodb -d  -p 27017:27017 mongo:3.2
```
- to list running containers:
```ruby 
docker ps -a  
```

===========


### Context Broker Installation

- to install orion context broker with docker and link it to mongodb:

```ruby 
sudo docker run --name orion -d -p 1026:1026 --link mongodb -h orion fiware/orion:latest -dbhost mongodb
```
- to see if it is working:
```ruby 
curl localhost:1026/version | python -mjson.tool mjson.tool mjson.tool mjson.tool
```
-Result (example):
```ruby 
ubuntu@ip-172-31-13-51:~$ curl localhost:1026/version
{
"orion" : {
  "version" : "2.4.0-next",
  "uptime" : "0 d, 0 h, 1 m, 13 s",
  "git_hash" : "4f26834ca928e468b091729d93dabd22108a2690",
  "compile_time" : "Tue Mar 31 16:21:23 UTC 2020",
  "compiled_by" : "root",
  "compiled_in" : "3369cff2fa4c",
  "release_date" : "Tue Mar 31 16:21:23 UTC 2020",
  "doc" : "https://fiware-orion.rtfd.io/"
}
}
```

## IoT Agent Installation

- CLONING THE GITHUB REPOSITORY:
```ruby
git clone https://github.com/Atos-Research-and-Innovation/IoTagent-LoRaWAN.git
```
- Download the dependencies for the project, and let it ready to the execution. From the root folder of the project execute:
```ruby
npm install
```
- Launch the IoT Agent with the default configuration:
```ruby
node bin/iotagent-lora
```

