This demo shows one of several different approaches to running Docker. This approach advertises host-routes for Docker containers which have been created on a macvlan.  No NAT is used.  Using this technique you can provide your containers with real IP addresses which are externally reachable and all ports are accepted.

In this solution, when containers existence are learned by the directly connected leaf switch, an ARP entry is added.  The leaf switch redistributes the ARP table into BGP, letting other leaf switches and spine switches receive a host route to the container.  The redistribute neighbor daemon running on the leaf switches continually sends a unicast ARP to all known containers to verify it's existence.   When containers are destroyed, the unicast ARPs no longer reply, the MAC/IP address is removed from the ARP table and the host routes are removed from the BGP advertisement.   

Using this technique you can deploy containers from a single 172.16.1.0/24 subnet owned by multiple docker macvlans on different hosts and located in different racks throughout the DC.  A tool must be used to prevent containers having duplicate IP addresses.  



Network Topology
----------------
![Network Topology for Docker-Macvlan demo ](https://github.com/CumulusNetworks/cldemo-docker-macvlan/blob/master/cldemo-docker-macvlan.png)



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
 - Cumulus v3.3.0

On Servers:
 - Ubuntu 16.04 
 - Docker-CE v17.05



Quickstart: Run the demo
------------------------



Before running this demo, install [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds) and [Vagrant](https://releases.hashicorp.com/vagrant/). The currently supported versions of VirtualBox and Vagrant can be found on the main [cldemo-vagrant](https://github.com/CumulusNetworks/cldemo-vagrant) documentation page in the "prequisites" section.

Once the prequisites have been installed, proceed with the steps below.

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    
    vagrant up oob-mgmt-server oob-mgmt-switch
    vagrant up leaf01 leaf02 leaf03 leaf04 spine01 spine02
    vagrant up server01 server02 server03 server04
    
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    
    git clone  https://github.com/CumulusNetworks/cldemo-docker-macvlan

    cd cldemo-docker-macvlan
    
    ansible-playbook run-demo.yml




Viewing the Results
-------------------



All the container /32 addresses can be seen:

From spine:

   
    cumulus@spine01:mgmt-vrf:~$ net show route
    
    show ip route
    =============
    Codes: K - kernel route, C - connected, S - static, R - RIP,
           O - OSPF, I - IS-IS, B - BGP, P - PIM, T - Table, v - VNC,
           V - VPN,
           > - selected route, * - FIB route
    B>* 10.0.0.11/32 [20/0] via fe80::4638:39ff:fe00:53, swp1, 00:32:57
    B>* 10.0.0.12/32 [20/0] via fe80::4638:39ff:fe00:28, swp2, 00:32:57
    B>* 10.0.0.13/32 [20/0] via fe80::4638:39ff:fe00:4f, swp3, 00:32:57
    B>* 10.0.0.14/32 [20/0] via fe80::4638:39ff:fe00:3b, swp4, 00:32:57
    C>* 10.0.0.21/32 is directly connected, lo
    B>* 172.16.1.11/32 [20/0] via fe80::4638:39ff:fe00:53, swp1, 00:22:14
    B>* 172.16.1.12/32 [20/0] via fe80::4638:39ff:fe00:53, swp1, 00:22:13
    B>* 172.16.1.13/32 [20/0] via fe80::4638:39ff:fe00:53, swp1, 00:22:13
    B>* 172.16.1.21/32 [20/0] via fe80::4638:39ff:fe00:28, swp2, 00:17:05
    B>* 172.16.1.22/32 [20/0] via fe80::4638:39ff:fe00:28, swp2, 00:17:05
    B>* 172.16.1.23/32 [20/0] via fe80::4638:39ff:fe00:28, swp2, 00:17:04
    B>* 172.16.1.31/32 [20/0] via fe80::4638:39ff:fe00:4f, swp3, 00:11:12
    B>* 172.16.1.32/32 [20/0] via fe80::4638:39ff:fe00:4f, swp3, 00:11:12
    B>* 172.16.1.33/32 [20/0] via fe80::4638:39ff:fe00:4f, swp3, 00:11:11
    B>* 172.16.1.41/32 [20/0] via fe80::4638:39ff:fe00:3b, swp4, 00:04:12
    B>* 172.16.1.42/32 [20/0] via fe80::4638:39ff:fe00:3b, swp4, 00:04:12
    B>* 172.16.1.43/32 [20/0] via fe80::4638:39ff:fe00:3b, swp4, 00:04:11
    
    show ipv6 route
    =================
    Codes: K - kernel route, C - connected, S - static, R - RIPng,
     O - OSPFv6, I - IS-IS, B - BGP, T - Table, v - VNC,
     V - VPN,
     > - selected route, * - FIB route
     C * fe80::/64 is directly connected, swp4
     C * fe80::/64 is directly connected, swp3
     C * fe80::/64 is directly connected, swp2
     C>* fe80::/64 is directly connected, swp1



Attaching to container from a server:

    vagrant@server01:~$ sudo docker exec -it 581764bbc3d9 /bin/bash
    root@581764bbc3d9:/# ip route show
    default via 172.16.1.1 dev eth0 
    172.16.1.0/24 dev eth0  proto kernel  scope link  src 172.16.1.12 

**Remember to use CTL P CTL Q to exit the container!**








`




