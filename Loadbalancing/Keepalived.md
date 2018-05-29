# Keepalived
> According to:  
> http://www.keepalived.org/doc/index.html  
> https://access.redhat.com/documentation/zh-tw/red_hat_enterprise_linux/7/html/load_balancer_administration/  

A frameworks for both load balancing and high availability.  
- Load balancing framework relies on Linux Virtual Server (IPVS) kernel module
- Provides Layer 4 load balancing

High availability is achieved by the Virtual Redundancy Routing Protocol (VRRP)  
- Two main functions:
    - Health checking for LVS systems
    - Implementation of the VRRP stack to handle load balancer failover

- 3 distinct processes:
    - A minimalistic parent process in charge with forked children process monitoring.
    - Two children processes
        - responsible for VRRP framework
        - responsible for healthchecking
```sh
PID         111     Keepalived  <-- Parent process monitoring children
            112     \_ Keepalived   <-- VRRP child
            113     \_ Keepalived   <-- Healthchecking child
```

## Kernel Components

1. LVS Framework: Uses the getsockopt and setsockopt calls to get and set options on sockets.
2. Netfilter Framework: IPVS code that supports NAT and Masquerading.
3. Netlink Interface: Sets and removes VRRP virtual IPs on network interfaces.
4. Multicast: VRRP advertisements are sent to the reserved VRRP MULTICAST group (224.0.0.18).

![](http://www.keepalived.org/doc/_images/software_design.png)
- Control Plane
    - configuration: keepalived.conf
    - compiler design:　parsing, translate keepalived.conf into an internal memory representation
- Scheduler - I/O Multiplexer
    - All the events are scheduled into the same process.
    - provides its own thread abstraction optimized for networking purpose
- Memory Management
    - two mode : normal_mode & debug_mode
- Core Components
    - defines some common and global libraries
        - html parsing, link-list, timer, vector, string formating, buffer dump, networking utils, daemon management, pid handling, low level TCP layer4
- WatchDog
    - A framework
    - provides children processes monitoring (VRRP & Healthchecking)
    - Each child accepts connection to its own watchdog unix domain socket
    - Parent process send “hello” messages to this child unix domain socket. Hello messages are sent using I/O multiplexer on the parent side and accepted/processed using I/O multiplexer on children side. If parent detects broken pipe it tests using sysV signal if child is still alive and restarts it.
- Checkers
    - one of the main Keepalived functionality
    - are in charge of realserver healthchecking
    - The internal checker design is realtime networking software, it uses a fully multi-threaded FSM design (Finite State Machine)
    - checker stack provides LVS topology manipulation accoring to layer4 to layer5/7 test results.
    - It’s run in an independent process monitored by parent process.
- VRRP Stack
    - one of the main Keepalived functionality
    - implements full IETF RFC2338 standard
    - extensions for LVS and Firewall design
- System Call
- Netlink Reflector
- SMTP
- IPVS Wrapper
- IPVS
- NETLINK
- Syslog
