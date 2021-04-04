# Netflow Collector v0.0.9
**NFCOL** Collects Netflow version 1, 5, 6, 7, 9 & IPFIX & stores them on **InfluxData** time-series DB (InfluxDB)


## Features
As I develop this tool for my personal usage at first step, It has small set of features:

- **Collect all available versions of Netflow**
  - **YAML** config file which helps you configure the whole application from a single file
  - Thanks to [tehmaze](https://github.com/tehmaze/netflow), We are using this repo to decode Netflow traffics & able to decode 1, 5, 6, 7, 9 & IPFIX protocols.
  - You can forward **decoded** Netflow packets to another hosts by enabling **Packet Forwarders** in **nfc.yaml** config file
    - In addition to packet forwarder, **nfc-dump** is a client for forwarded packets to print a nice tabular output. Also this tool can filter incomming packets according to the provided arguments.
  - Exporting traffics to **InfluxDB** time-series database.
    - To enable this feature you need to configure **nfc.yaml** config file
    - Also, It can provide you a GEO Location information for the SOURCE & DESTINATION IP Addresses (both IPv4 & IPv6).
      - To get more information about IP2Location tool you can read this GITHUB Repo. README file [IP2Location](https://github.com/javadmohebbi/IP2Location)


## Usage
In order to use this tool you can download the compiled binaries or you can compile it for yourself.
You can download the compiled versions from the further links:

### Download Binaries:
  - **Linux AMD64**:
    - *nfcol*: [Download Linux AMD64 Binaries](https://github.com/javadmohebbi/nfCollector/raw/master/dist/linux/amd64/nfcol)
    - *nfcol-dump*: [Download Linux AMD64 Binaries](https://github.com/javadmohebbi/nfCollector/raw/master/dist/linux/amd64/nfcol-dump)
  - **Windows AMD64**:
    - *nfcol.exe*: [Download Windows AMD64 Binaries](https://github.com/javadmohebbi/nfCollector/raw/master/dist/windows/amd64/nfcol.exe)
    - *nfcol-dump.exe*: [Download Windows AMD64 Binaries](https://github.com/javadmohebbi/nfCollector/raw/master/dist/windows/amd64/nfcol-dump.exe)
  - **MacOS AMD64**:
    - *nfcol*: [Download MacOS AMD64 Binaries](https://github.com/javadmohebbi/nfCollector/raw/master/dist/darwin/amd64/nfcol)
    - *nfcol-dump*: [Download MacOS AMD64 Binaries](https://github.com/javadmohebbi/nfCollector/raw/master/dist/darwin/amd64/nfcol-dump)



### Download Installation Packages
Currently I have just created a DEB package. I will provide RPM and MSI packages here soon ;-)

  - [Debian (deb) package for AMD64 bit systems](https://github.com/javadmohebbi/nfCollector/raw/master/dist/packages/amd64/deb/nfcol_0.0.9-amd64.deb)
    - To install this package run the command ```sudo dpkg -i nfcol_0.0.9-amd64.deb```
      - It will install nfcol, nfcol-dump and ip2location and will create a system service **nfcol.service**. Also it will download the latest IP2Location Lite DB from the Internet
      - Once Installation has completed, You can check the service status using ```sudo service nfcol status```       



## Prepare InfluxDB
If you want to export Netflow traffics to **InfluxDB** database you must install it. currently we support version 1.x (max version is 1.7)
- You can get InfluxDB installtion file from [This Link](https://portal.influxdata.com/downloads/)
- ***InfluxData recommends you to install InfluxDB on SSD*** and since we are going to store many metrics, SSD disk is recommended for better performance.
- After installing InfluxDB you can create a DB using this command: ```CREATE DATABASE "netflowDB" WITH DURATION 10d REPLICATION 1 SHARD DURATION 1h NAME "nfc"```


### Configuration
To config **nfcol** you need to provide configuration file in **yaml** format. ```*nix``` users must place this file in ```/etc/nfcol/nfc.yaml``` & ```windows``` users must place it in ```%ProgramFiles%\Netflow-Collector\nfc.yaml```
***If you use installtion packages, It will create it for you automatically***

Your ```nfc.yaml``` file must be look like this sample:
```
# # # # # # # # # # # # # # # # # #
#       Netflow Collector         #
#         Configuration           #
# # # # # # # # # # # # # # # # # #
server:
  # Listen Address
  address: 0.0.0.0

  # Listen UDP Port
  port: 6859

  # If true, nfc will write flow data into stdout
  dump: false

  # Activate forwarder
  forwarder: true

  # Host to forward - Can be separated by ; (semi-colon) eg: 127.0.0.1;192.168.100.1
  forwarderHost: 127.0.0.1

  # Forwarder UDP Port
  forwarderPort: 7161

# # # # # # # # # # # # # # # # # #
#    IP2Location Configuration    #
# # # # # # # # # # # # # # # # # #
ip2location:
  # IP2Location command path
  cmd: /usr/local/bin/ip2location

  # Path to Local GEO Database. Read more at:
  # https://github.com/javadmohebbi/IP2Location#local-database-format
  local: /etc/ip2location/local.csv

# # # # # # # # # # # # # # # # # #
#     Exporter Configuration      #
# # # # # # # # # # # # # # # # # #
exporter:
  # Enable if it's true
  enable: false

  # Currently Only InfluxDB (1.x) supported
  type: influxdb


# # # # # # # # # # # # # # # # # #
#     InfluxDB Configuration      #
# # # # # # # # # # # # # # # # # #
influxDB:
  # InfluxDB Host
  host: 127.0.0.1

  # InfluxDB Port
  port: 8086

  # InfluxDB Username. Can be null
  username: #user

  # InfluxDB Password. Can be null
  password: #secret

  # InfluxDB Database
  # InfluxDB command example for creating database:
  #       CREATE DATABASE "netflowDB" WITH DURATION 10d REPLICATION 1 SHARD DURATION 1h NAME "nfc"
  database: netflowDB

  # Temp Dir for InfluxDB Metrics. MUST be ended with / (Linux) or \ (Windows)
  tmpDir: /tmp/nfcol/


# # # # # # # # # # # # # # # # # #
#          Measurements           #
# # # # # # # # # # # # # # # # # #
measurements:
  # Netflow Summary Measurement Name
  summaryProto: sum_proto

  # Netflow GEO Summary Measurement Name
  summaryProtoGeo: sum_proto_geo
```


### IP2Location
If you want to use GEO location tool, you need to read it's usage at [IP2Location Github Repositories](https://github.com/javadmohebbi/IP2Location)



## Command line options
Here is the command line options for all binaries:
### nfcol command line options
```
  -addr string
        Listen IP address
  -debug string
        It will Print debug info if the value is 'true' and 'false' for nothing
  -dump string
        It will Print flow record if the value is 'true' and 'false' for nothing
  -port string
        Listen port
  -v    Print Version & exit.
```

- **addr**: Listen address for Netflow Collector tool. If nothing will be provided it will read it from **nfc.yaml** file
- **port**: Listen port for Netflow Collector tool. If nothing will be provided it will read it from **nfc.yaml** file
- **debug**: Enable debugging mode & provide more information about everything
- **dump**: Disable exporting & dump netflow traffics to Standard Output
- **h**: Show help and exit
- **v**: Print version and exit


### nfcol-dump command line options
This tool can connect to nfcol tools & **Filter** and **Print** Netflow Traffic to the standard output (Terminal, CMD, Powershell)
Almost all of options starts with **-flt** & accepts ***wildcards***.
```
  -addr string
        Listen IP address - Default 127.0.0.1
  -flt-dst-ip string
        Filter Destination IP. eg: 192.168.1.1, 192.168.1.* (default "*")
  -flt-dst-port string
        Filter Destination Port. eg: 80, 433, 100-250 (default "*")

  -flt-nf-exp string
        Filter netflow Exporter. IP address of exporter device. eg: 192.168.1.1, 192.168.1.* (default "*")
  -flt-nf-ver string
        Filter netflow version. eg: 1, 5, 6, 7, 9, 10 (for IPFIX) (default "*")
  -flt-proto string
        Filter Protocol. eg: tcp, udp, icmp (default "*")
  -flt-src-ip string
        Filter Source IP. eg: 192.168.1.1, 192.168.1.* (default "*")
  -flt-src-port string
        Filter Source Port. eg: 80, 433, 100-250 (default "*")
  -port string
        Listen port - Default 7161
```
- **addr**: Listen address for Netflow Dumper tool. If you want to listen on available IP addresses you need to provide ```0.0.0.0```
- **port**: Listen post for Netflow Dumper tool
- **flt-nf-ver**: Filter packet based on netflow version.
- **flt-nf-exp**: Filter packet based on netflow exporter. You might have different Netflow exporter and interested in one of them, So you can provide it here to filter the printer packets
- **flt-proto**: Filter based on protocols. Like tcp, udp, gre.
- **flt-src-ip** & **flt-dst-ip**: Filter based on source or destination IP Addresses.
- **flt-src-port** & **flt-dst-port**: Filter based on source or destination port.


# Grafana Dashboards
I have made two simple & powerfull **Grafana** dashboards which you can download them from the further links:

- [Netflow Exporter Overview](https://grafana.com/grafana/dashboards/11408)
- [Netflow Summary Overview](https://grafana.com/grafana/dashboards/11409)

![Netflow Exporter Dashboard](https://raw.githubusercontent.com/javadmohebbi/nfCollector/master/NetflowExporter%20Overview-Grafana.png)
![Netflow Summary Dashboard](https://raw.githubusercontent.com/javadmohebbi/nfCollector/master/NetflowSummary%20Overview-Grafana.png)
