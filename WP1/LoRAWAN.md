## <p align="center">  LoRaWAN Communication protocol</p>

## 1.  LoRaWAN specification

<p style='text-align: justify;'> LoRaWAN is an open Low Power Wide Area Network (LPWAN) standard developed by LoRa Alliance and has key features i.e., low power consumption, long-range communication, end-to-end 
security, mobility and localization services.</p>

<p style='text-align: justify;'> Network topology of the LoRaWAN is considered as star-of-stars and from the architectural point of view, the system has three main components: network servers, gateways (GWs) and end nodes. End nodes communicate with the network server via GWs and Node-to-GW communication can be either LoRa or FSK modulation with different data rates and channels. Network servers manage the GWs through standard IP technology and data frames sent through end nodes, received by GWs and routed through the network server.</p>








![png](images/Architecture.png)





LoRaWAN networks can be used in two ways:

• Using online services such as **the Things Network (TTN)** which offers a set of open tools to get up and run LoRaWAN:

https://www.thethingsnetwork.org/

• We can create our own private LoRaWAN network by using the **ChirpStack project components**.


In the following, we will install adn congifure the open source LoRaWAN ChirpStack project.



## 2.  ChirpStack project


1-ChirpStack is the new name for what was known as the LoRa Server project (this project is launched by Orne Brocaar in 2016).


2-The project consists of four major components: 

**-ChirpStack Gateway Bridge:**  abstracts the packet forwarder UDP protocol data into JSON and sends it to
the LoRa Server over MQTT.

**-ChirpStack Network Server:** is the LoRaWAN network server. It de-duplicates and handles the received uplink frames from
the gateway(s), handles the LoRaWAN mac layer and schedules the downlink data transmissions.

**-ChirpStack Application Server:** is the LoRaWAN Application Server and handles the join-requests, encryption of
application payloads and offers a RESTful JSON API, gRPC API or MQTT for external services. 

To visualize the received sensor data and manage the applocations, LoRa server provides a web interface.


**-ChirpStack Geolocation Server:** is an optional component which integrates with multiple LoRaWAN geolocation backends.

The LoRa Geo Server will not be discussed in this tutorial.



3-More information about ChirpStack project:


• ChirpStack project information:https://www.chirpstack.io/


• ChirpStack code:https://github.com/brocaar


• ChirpStack forum:https://forum.chirpstack.io/








## 3.  Considered ChirpStack architecture


The LoRa Server and the LoRa App Server each requires their own PostgreSQL
database.

 All ChirpStack components and their dependencies will be installed on the same own
server as shown in the following graph.






![png](images/output_8_0.png)



## 4.  LoRaWAN Implementation

## 4.1 Software requirements

    
    
   
    
    
    
    
    
            LoRa Gateway Bridge Server
    
    
Configuration file: /etc/lora-gateway-bridge/lora-gateway-bridge.toml

MQTT broker address and port: server=“tcp://127.0.0.1:1883”

        
            LoRa Network Server
            
Configuration file: /etc/loraserver/loraserver.toml

PostgreSQL settings: dsn=“postgres://loraserver_ns:dbpassword@localhost/loraserver_ns?sslmode=disable"
Automatically apply database migrations: automigrate=true
LoRaWAN regional band configuration: name=“EU_863_870"
System's local time zone: timezone=“Local”
MQTT broker address and port: server=“tcp://127.0.0.1:1883”

            LoRa App Server
            
         

Configuration file: /etc/lora-app-server/lora-app-server.toml
PostgreSQL settings: dsn=“postgres://loraserver_as:dbpassword@localhost/loraserver_as?sslmode=disable"
JWT secret: jwt_secret=“zHj0RGotdJbgtKpPrv0/2axdER4bsnss6SHueANdEUg="
MQTT broker address and port: server=“tcp://127.0.0.1:1883”
Internal API server: public_host=“localhost:8001”


**<font color='red'>Procedure</font>**


**<font color='green'>Step A: Install LoRa Gatway Bridge</font>** 


    LoRa Gateway Bridge makes use of MQTT for publishing and receiving application payloads.
    Type: sudo apt-get install mosquitto

    The LoRa Server project provides pre-compiled binaries packaged as Debian (.deb) packages.
    In order to activate this repository, execute the following commands:
    Type: sudo apt install apt-transport-https dirmngr
    Type: sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
    Type: sudo echo "deb https://artifacts.loraserver.io/packages/2.x/deb stable main" | sudo tee /etc/apt/sources.list.d/loraserver.list
    Type: sudo apt-get update

    Install the LoRa Gateway Bridge.
    Type: sudo apt-get install lora-gateway-bridge

    Note 1:
    This will setup an user and group, create start scripts for systemd or init.d
    (this depends on your version of Debian / Ubuntu).
    Note 2:
    lora-gateway-bridge configuration file is installed at: /etc/lora-gateway-bridge/lora-gateway-bridge.toml
    lora-gateway-bridge executable is installed at: /usr/bin/lora-gateway-bridge
    Note 3:
    More information: /usr/bin/lora-gateway-bridge --help

    Start LoRa Gateway Bridge.
    Type: sudo systemctl start lora-gateway-bridge

    If your distribution uses systemd or init.d, use one of the following commands:
    Note: Raspberry Pi uses systemd.
    [systemd] Type: sudo systemctl [start|stop|restart|status] lora-gateway-bridge
    [init.d] Type: sudo /etc/init.d/lora-gateway-bridge [start|stop|restart|status]

    Verify if the LoRa Gateway Bridge is running and check if there are no errors.
    Type: systemctl status lora-gateway-bridge

    Check LoRa Gateway Bridge log output.
    [systemd] Type: journalctl -f -n 50 -u lora-gateway-bridge
    [init.d] Type: tail -f -n 50 /var/log/lora-gateway-bridge/lora-gateway-bridge.log

    Start lora-gateway-bridge on boot.
    Type: sudo systemctl enable lora-gateway-bridge

    Note:
    To disable the LoRa Gateway Bridge so it doesn't start on boot.
    Type: sudo systemctl disable lora-gateway-bridge



**<font color='green'>Step B: Install Network Application Server</font>** 

    Install PostgreSQL.
    Type: sudo apt-get install postgresql

    Install Redis server.
    Type: sudo apt-get install redis-server

    LoRa Server needs its own database.
    To create a new database, start the PostgreSQL prompt as the postgres user.
    Type: sudo -u postgres psql

    Within the PostgreSQL prompt, enter the following queries:
    Type: create role loraserver_ns with login password 'dbpassword';

    Note: The database password is: 'dbpassword'
    Change this to something more secure.

    Type: create database loraserver_ns with owner loraserver_ns;
    Type: \q

    Verify if the user and database have been setup correctly, try to connect to it:
    Type: psql -h localhost -U loraserver_ns -W loraserver_ns
    Type: \q

    Install LoRa Server.
    Type: sudo apt-get install loraserver

    Note 1:
    loraserver configuration file is installed at: /etc/loraserver/loraserver.toml
    loraserver executable is installed at: /usr/bin/loraserver
    Note 2:
    More information: /usr/bin/loraserver --help

    Modify the LoRa Server configuration file /etc/loraserver/loraserver.toml.
    Type: sudo su
    Type: cd /etc/loraserver
    Type: nano loraserver.toml

    Make the following changes (if needed):
    dsn="postgres://loraserver_ns:dbpassword @localhost/loraserver_ns?sslmode=disable"
    automigrate=true
    name="EU_863_870"
    timezone="Local"
    server="tcp://127.0.0.1:1883"

    Note 1: dsn
    Given you used the password dbpassword when creating the PostgreSQL database.
    Note 2: automigrate
    Automatically apply database migrations.
    LoRa App Server migrate to the latest state automatically.
    Note 3: name
    LoRaWAN regional band configuration. Valid values:
    AS_923
    AU_915_928
    CN_470_510
    CN_779_787
    EU_433
    EU_863_870
    IN_865_867
    KR_920_923
    RU_864_870
    US_902_928
    Note 4: timezone
    The timezone is used. Example: "Europe/Amsterdam" or "Local" for the the system's local time zone.
    Note 5: server
    MQTT broker address and port

    Type: exit (After changes)

    Start LoRa Server.
    Type: sudo systemctl start loraserver

    If your distribution uses systemd or init.d, use one of the following commands:

    Note: Raspberry Pi uses systemd.
    [systemd] Type: sudo systemctl [start|stop|restart|status] loraserver
    [init.d] Type: sudo /etc/init.d/loraserver [start|stop|restart|status]

    Verify if the LoRa Server is running.
    Type: systemctl status loraserver

    Check LoRa Server log output.
    [systemd] Type: journalctl -f -n 50 -u loraserver
    [init.d] Type: tail -f -n 50 /var/log/loraserver/loraserver.log

    Start LoRa Server on boot.
    Type: sudo systemctl enable loraserver

    Note:
    To disable the LoRa Server so it doesn't start on boot.
    Type: sudo systemctl disable loraserver



**<font color='green'>Step C: Install Application server</font>** 

    LoRa App Server persists the gateway data into a PostgreSQL database.
    LoRa App Server needs its own database.
    To create a new database, start the PostgreSQL prompt as the postgres user:
    Type: sudo -u postgres psql

    Within the PostgreSQL prompt, enter the following queries:
    Type: create role loraserver_as with login password 'dbpassword';
    Type: create database loraserver_as with owner loraserver_as;

    Enable the pg_trgm (trigram) extension.
    Type: \c loraserver_as
    Type: create extension pg_trgm;
    Type: \q

    Verify if the user and database have been setup correctly, try to connect to it:
    Type: psql -h localhost -U loraserver_as -W loraserver_as
    Type: \q

    Install LoRa App Server.
    Type: sudo apt-get install lora-app-server

    Note 1:
    lora-app-server configuration file is installed at: /etc/lora-app-server/lora-app-server.toml
    lora-app-server executable is installed at: /usr/bin/lora-app-server
    Note 2:
    More information: /usr/bin/lora-app-server --help

    Create a JSON Web Token (jwt). Open a terminal.
    Type: openssl rand -base64 32
    For example: e3+eD7zcVFJF3EFpPnM1oMj02DqUZxt5wR4IfPBpbtA=
    Copy and save the output. Will be used at the next step. Keep this secret!

    Modify the lora-app-server configuration file.
    /etc/lora-app-server/lora-app-server.toml.
    Type: sudo su
    Type: cd /etc/lora-app-server
    Type: nano lora-app-server.toml

    Make the following changes (if needed):
    dsn="postgres://loraserver_as:dbpassword@ localhost/loraserver_as?sslmode=disable"
    jwt_secret="e3+eD7zcVFJF3EFpPnM1oMj02DqUZxt5wR4IfPBpbtA="
    server="tcp://localhost:1883"
    public_host="localhost:8001"

    Note 1: dsn
    Given you used the password dbpassword when creating the PostgreSQL database.
    Note 2: jwt_secret
    jwt_secret, see step 6.
    Note 3: server
    MQTT broker address and port
    Note 4: public_host
    The Internal API Server is used by LoRa Server to communicate with LoRa App Server

    Type: exit (After changes)

    Start LoRa App Server.
    Type: sudo systemctl start lora-app-server

    If your distribution uses systemd or init.d, use one of the following commands:
    Note: Raspberry Pi uses systemd.
    [systemd] Type: sudo systemctl [start|stop|restart|status] lora-app-server
    [init.d] Type: sudo /etc/init.d/lora-app-server [start|stop|restart|status]

    Verify if the LoRa App Server is running.
    Type: systemctl status lora-app-server

    Check LoRa App Server log output.
    [systemd] Type: journalctl -f -n 50 -u lora-app-server
    [init.d] Type: tail -f -n 50 /var/log/lora-app-server/lora-app-server.log

    Start LoRa App Server on boot.
    Type: sudo systemctl enable lora-app-server

    Note:
    To disable the LoRa App Server so it doesn't start on boot.
    Type: sudo systemctl disable lora-app-server


Step F. Check if all required services are running

    Type: systemctl status ttn-gateway
    Type: systemctl status mosquitto
    Type: systemctl status lora-gateway-bridge
    Type: systemctl status loraserver
    Type: systemctl status lora-app-server

Step G. LoRa App Server Web Interface

    Open a web browser and enter the IP address where the LoRa App Server is installed,
    followed by port 8080. In this manual the LoRa App Server is installed on the Gateway.
    For example: http://192.168.1.71:8080

    Login with the default credentials:
    user: admin
    password: admin

    Select menu: Network-servers
    Press button: ADD
    Tab: GENERAL
    Network-server name: LoRaServer1
    Network-server server: localhost:8000
    Note 1: Use localhost, if LoRa Server is installed on the same host as LoRa App Server.
    Note 2: The LoRa App Server can connect to multiple LoRa Server instances.

    Tab: GATEWAY DISCOVERY
    Uncheck: Enable gateway discovery

    Note 3: In this manual I assume you only have one gateway.
    Therefore gateway discovery is disabled.

    Select: ADD NETWORK-SERVER or UPDATE NETWORK-SERVER

    Select menu: Gateway-Profiles
    Nothing done

    Select menu: Organizations
    Select: loraserver

    Change Organization name: mobilefish
    The name may only contain words, numbers and dashes.

    Display name: Mobilefish Research and Development

    Check: Organization can have gateways
    When checked, it means that organization administrators are able to add their own gateways to the network. Note that the usage of the gateways is not limited to this organization.

    Select: UPDATE ORGANIZATION

    Select menu: Service-Profiles

    Note:
    A service-profile connects an organization to a network-server and defines the features that an organization can use on this network-server.

    Press button: CREATE

    Service-profile name: ServiceProfile1
    A name to identify the service-profile.

    Network-server: LoRaServer1
    The network-server on which this service-profile will be provisioned. After creating the service-profile, this value can't be changed.

    Check: Add gateway meta-data
    GW metadata (RSSI, SNR, GW geoloc., etc.) are added to the packet sent to the application-server.

    Uncheck: Enable network geolocation
    When enabled, the network-server will try to resolve the location of the devices under this service-profile. Please note that you need to have gateways supporting the fine-timestamp feature and that the network-server needs to be configured in order to provide geolocation support.

    Device-status request frequency: 0
    Frequency to initiate an End-Device status request (request/day). Set to 0 to disable.

    Minimum allowed data-rate: 0
    Minimum allowed data rate. Used for ADR. This applies for the EU.

    Maximum allowed data-rate: 5
    Maximum allowed data rate. Used for ADR. This applies for the EU.

    Select: CREATE SERVICE-PROFILE

    Select menu: Device-Profiles

    Note:
    A device-profile defines the capabilities and boot parameters of a device. You can create multiple device-profiles for different kind of devices.

    Press button: CREATE

    Device-profile name: DEVPROF-EU868
    A name to identify the device-profile.

    Network-server: LoRaServer1
    The network-server on which this device-profile will be provisioned.
    After creating the device-profile, this value can't be changed.

    LoRaWAN MAC version: 1.0.2
    The LoRaWAN MAC version supported by the device.
    I will use the Arduino LMIC Library when I upload my sketch (Step H).
    https://github.com/matthijskooijman/arduino-lmic

    The Arduino LMIC Library supports LoRaWAN MAC version 1.0.2 revision A
    See also:
    https://forum.loraserver.io/t/generic-arduino-lmic-based-devices/2991

    LoRaWAN Regional Parameters revision: A
    Revision of the Regional Parameters specification supported by the device.

    MAX EIRP: 0

    Select: CREATE DEVICE-PROFILE

    Select Device-profile: DeviceB
    Select tab: JOIN (OTAA/ABP)
    Check: Device supports OTAA
    Select tab: Class-B
    Uncheck: Device supports Class-B (I have disabled this because I have not done any testing with Class B before)
    Select tab: Class-C
    Uncheck: Device supports Class-C (I have disabled this because I have not done any testing with Class C before)

    Select: UPDATE DEVICE-PROFILE

    Select menu: Gateways
    Press button: CREATE

    Gateway name: RAK831
    The name may only contain words, numbers and dashes.

    Gateway description: RAK831 Pilot Gateway

    Gateway ID: B827EBFFFEC74B36

    Note:
    The gateway ID can be found in the local_conf.json file.
    In the RAK831 Pilot Gateway this file can be found at:
    /opt/ttn-gateway/packet_forwarder/lora_pkt_fwd

    https://github.com/robertlie/RAK831-LoRaGateway-RPi/
    blob/master/configuration_files/local_conf.json#L3

    Network-server: LoRaServer1
    Select the network-server to which the gateway will connect. When no network-servers are available in the dropdown, make sure a service-profile exists for this organization.

    Gateway-Profile:
    An optional gateway-profile which can be assigned to a gateway. This configuration can be used to automatically re-configure the gateway when LoRa Gateway Bridge is configured so that it manages the packet-forwarder configuration.

    Uncheck: Gateway discovery enabled
    When enabled (and LoRa Server is configured with the gateway discover feature enabled), the gateway will send out periodical pings to test its coverage by other gateways in the same network.

    Gateway altitude (meters): 10
    When the gateway has an on-board GPS, this value will be set automatically when the network received statistics from the gateway.

    Gateway location: <drag the marker to the location of the gateway> Drag the marker to the location of the gateway. When the gateway has an on-board GPS, this value will be set automatically when the network receives statistics from the gateway.

    Select: CREATE GATEWAY

    Select menu: Applications
    Press button: CREATE

    Application name: AppB
    The name may only contain words, numbers and dashes.

    Application description: AppB

    Service-profile: ServiceProfile1
    The service-profile to which this application will be attached.
    Note that you can't change this value after the application has been created.

    Payload codec: Custom Javascript codec functions
    By defining a payload codec, LoRa App Server can encode and decode the binary device payload for you.

    I have entered a Decode function which is based on an end node and sketch which is explained in step H.

    Enter the following:
    // Decode decodes an array of bytes into an object.
    // - fPort contains the LoRaWAN fPort number
    // - bytes is an array of bytes, e.g. [225, 230, 255, 0]
    // The function must return an object, e.g. {"temperature": 22.5}

    function Decode(fPort, bytes) {
       if(bytes.length == 1) {
          if(bytes[0] == 1) {
             return {
                'button': 'activated'
             }
          } else {
             return {
                'error': 'button action unknown'
             }
          }
       } else if(bytes.length == 4) {
          var humidity = (bytes[0]<<8) | bytes[1];
          var temperature = (bytes[2]<<8) | bytes[3];
             return {
                'humidity': humidity/ 100,
                'temperature': temperature/100
             }
          } else {
             return {
                'error': 'payload unknown'
             }
          }
       }
    }

    Select: CREATE APPLICATION

    Select: App1
    Select tab: DEVICES
    Press button: CREATE

    Device name: TESTDEVICE1
    The name may only contain words, numbers and dashes.

    Device description: TESTDEVICE1

    Device EUI: <generate_a_device_eui>
    Generate a new Device EUI. Make sure you select LSB!
    Copy the generated Device EUI, you will need it in step H.
    For example: c6 50 16 27 67 60 64 11 (LSB)
    This is the same as "DEVEUI" in the Arduino sketch.

    Device-profile: DEVPROF-EU868

    Check: Disable frame-counter validation
    Note that disabling the frame-counter validation will compromise security as it enables people to perform replay-attacks.

    Select tab: KEYS (OTAA)

    Application key (LoRaWAN 1.0): <generate_an_application_key>
    Generate a new Application Key. Make sure you select MSB!
    Copy the generated Application Key, you will need it in step H.
    For example: eb f3 a6 6d 69 66 6b 31 e2 5f 9a 13 29 44 6b 4a (MSB)
    This is the same as "APPKEY" in the Arduino sketch.

    Press button: SET DEVICE-KEYS

Step H. Modify and upload a sketch to the end node.

    I have built my own LoRa end node using an Arduino Pro Mini and HopeRF RFM95 LoRa transceiver module, see:
    https://www.mobilefish.com/developer/lorawan/ lorawan_quickguide_build_lora_node_rfm95_arduino_pro_mini.html

    This LoRa end node consist of:
        Arduino Pro Mini (ATmega328P / 3.3V / 8 MHz)
        LoRa HopeRF RFM95 module with dipole antenna
        DHT11 module (Humidity and temperature sensor)
        Push button
        Two LEDs
        OLED display module (0.96")

    I am using this Arduino sketch:
    https://www.mobilefish.com/download/lora/lmic-ttn-otaa-pro-mini-sensors-oled.ino.txt

    In this sketch the end node uses OTAA to register itself.

    Modify this sketch.
        Change the DEVEUI. Enter the Device EUI from previous step!
        static const u1_t PROGMEM DEVEUI[8]={ 0xC6, 0x50, 0x16, 0x27, 0x67, 0x60, 0x64, 0x11 };

        Change the APPKEY. Enter the Application Key from previous step!
        static const u1_t PROGMEM APPKEY[16] = { 0xEB, 0xF3, 0xA6, 0x6D, 0x69, 0x66, 0x6B, 0x31, 0xE2, 0x5F, 0x9A, 0x13, 0x29, 0x44, 0x6B, 0x4A };

        LoRaServer does not use the APPEUI value.
        You can use any value, for example:
        static const u1_t PROGMEM APPEUI[8]={ 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };

    Upload the sketch to the device.

Step I. Check if sensor data is displayed in the LoRa App Server web interface.

    Login
    Select menu: Applications
    Select: AppB
    Select Device: TESTDEVICE1
    Select menu: LIVE DEVICE DATA

Step J. Check the lora-gateway-bridge, loraserver and lora-app-server for errors.
This step is only needed if you encounter problems.

    Check if packet forwarder sends data:

    Type: sudo tcpdump -AUq -i lo port 1700
    Type: journalctl -f -n 50 -u lora-gateway-bridge
    Type: journalctl -f -n 50 -u loraserver
    Type: journalctl -f -n 50 -u lora-app-server

    More troubleshooting:
    https://www.loraserver.io/guides/troubleshooting/gateway/


    


```python

```
