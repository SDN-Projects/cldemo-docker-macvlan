This demo shows one of several different approaches to running Docker. This approach advertises host-routes for Docker containers which have been created on a macvlan.  No NAT is used.  Using this technique you can provide your containers with real IP addresses which are externally reachable and all ports are accepted.

In this solution, when containers existence are learned by the directly connected leaf switch, an ARP entry is added.  The leaf switch redistributes the ARP table into BGP, letting other leaf switches and spine switches receive a host route to the container.  The redistribute neighbor daemon running on the leaf switches continually sends a unicast ARP to all known containers to verify it's existence.   When containers are destroyed, the unicast ARPs no longer reply, the MAC/IP address is removed from the ARP table and the host routes are removed from the advertisement.   

Using this technique you can deploy containers from a single 172.16.1.0/24 subnet owned by multiple docker macvlans on different hosts and located in different racks throughout the DC.  A tool must be used to prevent containers having duplicate IP addresses.  



Network Topology
----------------
![Network Topology for Docker-Macvlan demo ](https://github.com/Diane-cumulus/cldemo-docker-macvlan/blob/master/cldemo-docker-macvlan.png)



----------


This is the topology in use in this demo. Three containers are hosted on each server as shown. The leaf switch runs redistribute neighbor daemon and redistributes the ARP table into BGP. The routes then get propagated into the core.   

This demo deploys 12 containers, three on each of the four servers.  

12 "workload" containers (3 per Server) -- The containers run our workloads at different unique IP Addresses all within the same /24 subnet.  In this demo, each server is on a different rack.

 - Server01 -- 172.16.1.11/24, 172.16.1.12/24, 172.16.1.13/24 
 - Server02 -- 172.16.1.21/24, 172.16.1.22/24, 172.16.1.23/24 
 - Server03 -- 172.16.1.31/24, 172.16.1.32/24, 172.16.1.33/24
 - Server04 -- 172.16.1.41/24, 172.16.1.42/24, 172.16.1.43/24

Each workload container identifies itself by sending a ping to it's local leaf switch interface.


Software in Use for this Demo
-----------------------------



On Spines and Leafs:
 - Cumulus v3.2.0

On Servers:
 - Ubuntu 16.04 
 - Docker-CE v17.05



Quickstart: Run the demo
------------------------



Before running this demo, install [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds) and [Vagrant](https://releases.hashicorp.com/vagrant/). The currently supported versions of VirtualBox and Vagrant can be found on the main [cldemo-vagrant](https://github.com/CumulusNetworks/cldemo-vagrant) documentation page under the "prequisites" section.

Once the prequisites have been installed, proceed with the steps below.

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    
    vagrant up oob-mgmt-server oob-mgmt-switch
    vagrant up leaf01 leaf02 leaf03 leaf04 spine01 spine02
    vagrant up server01 server02 server03 server04
    
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    
    git clone  https://github.com/CumulusNetworks/cldemo-docker-macvlan

    ansible-playbook run-demo.yml




Viewing the Results
-------------------



All the container /32 addresses can be seen:

From spine:

    vagrant@spine01:~$ ip route show
    default via 192.168.0.254 dev eth0 
    10.0.0.11 via 169.254.0.1 dev swp1  proto zebra  metric 20 onlink 
    10.0.0.12 via 169.254.0.1 dev swp2  proto zebra  metric 20 onlink 
    10.0.0.13 via 169.254.0.1 dev swp3  proto zebra  metric 20 onlink 
    10.0.0.14 via 169.254.0.1 dev swp4  proto zebra  metric 20 onlink 
    172.16.1.11 via 169.254.0.1 dev swp1  proto zebra  metric 20 onlink 
    172.16.1.12 via 169.254.0.1 dev swp1  proto zebra  metric 20 onlink 
    172.16.1.13 via 169.254.0.1 dev swp1  proto zebra  metric 20 onlink 
    172.16.1.21 via 169.254.0.1 dev swp2  proto zebra  metric 20 onlink 
    172.16.1.22 via 169.254.0.1 dev swp2  proto zebra  metric 20 onlink 
    172.16.1.23 via 169.254.0.1 dev swp2  proto zebra  metric 20 onlink 
    172.16.1.31 via 169.254.0.1 dev swp3  proto zebra  metric 20 onlink 
    172.16.1.32 via 169.254.0.1 dev swp3  proto zebra  metric 20 onlink 
    172.16.1.33 via 169.254.0.1 dev swp3  proto zebra  metric 20 onlink 
    172.16.1.41 via 169.254.0.1 dev swp4  proto zebra  metric 20 onlink 
    172.16.1.42 via 169.254.0.1 dev swp4  proto zebra  metric 20 onlink 
    172.16.1.43 via 169.254.0.1 dev swp4  proto zebra  metric 20 onlink 
    192.168.0.0/24 dev eth0  proto kernel  scope link  src 192.168.0.21 


Attaching to container from a server:

    vagrant@server01:~$ sudo docker exec -it 581764bbc3d9 /bin/bash
    root@581764bbc3d9:/# ip route show
    default via 172.16.1.1 dev eth0 
    172.16.1.0/24 dev eth0  proto kernel  scope link  src 172.16.1.12 

**Remember to use CTL P CTL Q to exit the container!**








`




