# cldemo-docker-macvlan

Advertising Containers Host Addresses with Redistribute Neighbor

This demo shows one of several different approaches to running Docker. This approach advertises host-routes for Docker containers which have been created on a macvlan.  No NAT is used.  Using this technique you can provide your containers with real IP addresses which are externally reachable.

 When containers are destroyed, the daemon also removes the host-routes from the fabric.

Using this technique you can deploy containers from a single large 172.16.0.0/16 subnet owned by multiple docker bridges on different hosts and located in different racks throughout the DC.

Network Topology

Network Topology

This is the topology in use in this demo. Three containers are hosted on each server as shown. The leaf switch learns the containers IP address through ARP, and redistributes the ARP table into BGP.  The routes then get propagated into the core.   

After the demo has been deployed 12 containers will have been deployed.


12 "workload" containers (4 per Server) -- This container runs our workloads at different unique IP Addresses all within the same /24 subnet.
Server01 -- 172.16.1.11, 172.16.2.12, 172.16.3.13
Server02 -- 172.16.1.21, 172.16.2.22, 172.16.3.23
Server03 -- 172.16.1.31, 172.16.2.32, 172.16.3.33
Server04 -- 172.16.1.41, 172.16.2.42, 172.16.3.43
