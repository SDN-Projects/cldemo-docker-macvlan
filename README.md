# cldemo-docker-macvlan

Advertising Containers Host Addresses with Redistribute Neighbor

This demo shows one of several different approaches to running Docker. This approach advertises host-routes for Docker containers which have been created on a macvlan.  No NAT is used.  Using this technique you can provide your containers with real IP addresses which are externally reachable.

In this solution, when containers existence are learned by the directly connected leaf switch, an ARP entry is added.  The leaf switch redistributes the ARP table into BGP, letting other leaf switches and spine switches receive a host route to the container.  The redistribute neighbor daemon running on the leaf switches continually sends a unicast ARP to all known containers to verify it's existence.   When containers are destroyed, the unicast ARPs no longer reply, the MAC/IP address is removed from the ARP table and the host routes are removed from the advertisement.   

Using this technique you can deploy containers from a single 172.16.1.0/24 subnet owned by multiple docker macvlans on different hosts and located in different racks throughout the DC.

Network Topology

Network Topology

This is the topology in use in this demo. Three containers are hosted on each server as shown. The leaf switch runs redistribute neighbor daemon and redistributes the ARP table into BGP. The routes then get propagated into the core.   

This demo deploys 12 containers, three on each of the four servers.  

Since we use a standard apache container that does not enable GARP, you must log into each leaf switch and ping each container to populate the leaf's arp table.  From then on, the ARP table should remain populated until the container is stopped.  


12 "workload" containers (4 per Server) -- This container runs our workloads at different unique IP Addresses all within the same /24 subnet.
Server01 -- 172.16.1.11, 172.16.1.12, 172.16.1.13
Server02 -- 172.16.1.21, 172.16.1.22, 172.16.1.23
Server03 -- 172.16.1.31, 172.16.1.32, 172.16.1.33
Server04 -- 172.16.1.41, 172.16.1.42, 172.16.1.43
