# consul-geo-failover

## Repo description
This repo provides Vagrant development environment demonstrating consul **geo failover** using **prepared queries**. In order to demonstrate this, we have simple configuration of 2 Consul Servers located in 2 different datacenters. Every Consul server has 2 Clients.  


Geo Failover function purpuse is to find where specific service is running. In this case we will search by **tag**


Some more details:
- Consul is configured as a systemd service.
- There is configured non-priviliged user called **consul**, which purpose is to run consul.
- Servers are configured with bootstrap enabled.
- Both clients have simple [web service](https://github.com/berchev/consul-geo-failover/blob/master/scripts/client_service.sh) enabled 
- All nodes have syslog logging enabled
- All nodes have configured data directory and configuration directory
- Servers have **retry_join_wan** in order to accomplish communication between the datacenters

For more details related to Consul Server/Client configuration, you can check [server_provision.sh](https://github.com/berchev/consul-geo-failover/blob/master/scripts/server_provision.sh) and [client_provision.sh](https://github.com/berchev/consul-geo-failover/blob/master/scripts/client_provision.sh) scripts.

This project can be used as a fundamental step for other consul related project.

## Repo content
| File                   | Description                      |
|         ---            |                ---               |
| [Vagrantfile](Vagrantfile) | Vagrant template file. TFE env is going to be cretated based on that file|
| [scripts/server_provision.sh](scripts/server_provision.sh) | Consul Server provision script|
| [scripts/client_provision.sh](scripts/client_provision.sh) | Consul Client provision script|
| [scripts/client_service.sh](scripts/client_service.sh) | Consul Client web service creation script|
| [payload.json](payload.json)| static failover policy including a fixed list of datacenters|


## Requirements
- VirtualBox installed
- Hashicorp Vagrant installed

## How to use this project
- clone the repo 
```
git clone https://github.com/berchev/consul-geo-failover.git
```
- change to repo directory
```
cd consul-geo-failover
```
- start provisioning of vagrant environment
```
vagrant up
```
- verify that all servers and client are in running status
```
vagrant status
```
- output should look like this:
```
Current machine states:

server-dc1                running (virtualbox)
client1-dc1               running (virtualbox)
client2-dc1               running (virtualbox)
server-dc2                running (virtualbox)
client1-dc2               running (virtualbox)
client2-dc2               running (virtualbox)


This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
- ssh to node `server-dc1`
```
vagrant ssh consul-server1
```
- change to `/vagrant` directory
```
cd /vagrant
```
- create a prepared query based on `payload.json` failover policy. (query will look for service called `web`, with tag `v1.1.1`)
```
curl http://127.0.0.1:8500/v1/query --request POST --data @payload.json
```
- output should be similar to this one:
```
{"ID":"095995a4-2738-4e02-715d-1030e251d1d6"}
```
- checking with `dig` whether query is working as expected
```
dig @127.0.0.1 -p 8600 web.query.consul +short
```
- the output should be the IP of the client, where this service is running. In our case:
```
192.168.51.21
```
- you can check the GUI interface for better visibility. Type `http://192.168.51.11:8500` in your favourite browser and will see very intuitive user friendly interface.
