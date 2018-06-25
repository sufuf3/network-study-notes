# Host Extensions for IP Multicasting
> 主機擴展用於IP多點傳送

- Ref:
    - https://tools.ietf.org/html/rfc1112
    - https://www.juniper.net/documentation/en_US/junos/topics/reference/standards/multicast-ip.html

A host implementation of the Internet Protocol (IP) to support multicasting  
> 自己理解的中文好記法：Sender 傳一個封包，但到 multicast router 會使封包分裂(如同細胞分裂的概念)，最後送到在同一個 group 的多個 hosts。  

## 介紹
IP multicasting 就是將 IP datagram 傳送到 "host group"。"host group" 就是一組 zero or more hosts identified by **a single IP destination address**.  
A multicast datagram is delivered to all members of its destination host group with the same "best-efforts" reliability as regular unicast IP datagrams.  
The datagram 不能保證完整地送到 all members of the destination group or in the same order relative to other datagrams.  
- The membership of a host group is dynamic(hosts 可以自由的加入或離開 groups)。
    - group 沒有 hosts 的數量限制
    - host 可以同時隸屬一個或多個 group。
    - host 在送 datagrams 的時候，可以不用成為 group 的 menber.
- A host group 可以是永久(permanent)或暫時(transient)的
    - permanent group: has a well-known, administratively assigned IP addr. (The IP addr not the membership of the group)
    - transient group: The IP multicast addrs that are not reserved for permanent groups are available for dynamic assignment to transient groups which exist only as long as they have members.

- IP multicast datagrams forwarding
    - is handled by "multicast routers" (co-resident with, or separate from, internet gateways)  
當一個 host 在 local network multicast 中傳送一個 IP multicast datagram 它可以到達所有緊鄰的鄰居的 destination host group 成員。  
如果 datagram 的 IP TTL > 1， 則連接到 local network 的 multicast router，讓 router 負責將其轉送到其他 Network 的 destination group 成員。  
在 IP TTL 內到達的那些 member networks ，an attached multicast router completes delivery by transmitting the datagram as a local multicast.  
  
Specifies the extensions required of a host IP implementation to support IP multicasting  
A "host" is any internet host or gateway other than those acting as multicast routers.  
The algorithms and protocols used within and between multicast routers are transparent to hosts and will be specified in separate documents.  

## Three levels of conformance
- Level 0: no support for IP multicasting.
- Level 1: support for sending but not receiving multicast IP datagrams.
- Level 2: full support for IP multicasting.
    - It requires implementation of the Internet Group Management Protocol (IGMP) 
    - It requires implementation of extension of the IP and local network service interfaces within the host.

## Host group addresses
Be identified by:  
    - class D IP address: with "1110" as their high-order four bits
    - Class E IP address: with "1111" as their high-order four bits, are reserved for future addressing modes
- host group addresses range from 224.0.0.0 to 239.255.255.255.
    - The address 224.0.0.0 is guaranteed not to be assigned to any group
    - 224.0.0.1 is assigned to the permanent group of all IP hosts (including gateways)
    - https://en.wikipedia.org/wiki/Multicast_address

## Model of a host IP implementation
In this model, ICMP and (for level 2 hosts) IGMP are considered to be implemented within the IP module, and the mapping of IP addresses to local network addresses is considered to be the responsibility of local network modules.  
Only  for expository purposes only  
```
         |                                                          |
         |              Upper-Layer Protocol Modules                |
         |__________________________________________________________|

      --------------------- IP Service Interface -----------------------
          __________________________________________________________
         |                            |              |              |
         |                            |     ICMP     |     IGMP     |
         |             IP             |______________|______________|
         |           Module                                         |
         |                                                          |
         |__________________________________________________________|

      ---------------- Local Network Service Interface -----------------
          __________________________________________________________
         |                            |                             |
         |           Local            | IP-to-local address mapping |
         |          Network           |         (e.g., ARP)         |
         |          Modules           |_____________________________|
         |      (e.g., Ethernet)                                    |
         |                                                          |
```

## Sending Multicast IP datagrams

### 1. Extensions to the IP Service Interface
使用相同的 "Send IP" operation 傳送 unicast IP datagrams 的方式來達到傳送 Multicast IP datagrams。  
An upper-layer protocol module 是指定一個 IP host group address 當作是目的主機。  
However, a number of extensions may be necessary or desirable.  

1. service interface 必須提供 upper-layer protocol 來指定 an outgoing multicast datagram 的 IP TTL 方式。  
如果 upper-layer protocol 沒有指定 TTL ，則 TTL 預設是 1。  
2. service interface 必須提供 upper-layer protocol 來指定哪一個是 multicast transmission 的 network interface 方式，當對於可能連接到多個 network 的 host。  
初始傳輸僅使用一個 interface ; 如有必要，multicast routers 會負責轉發到任何其他 networks。 如果 upper-layer protocol 選擇不識別出 interface ，則應該使用 default interface。  
3. (Only for level 2) service interface 必須提供 upper-layer protocol 來禁止 datagram 的本地傳送方式，當 Sender 的 host 是 group 中的成員。  
by default, a copy of the datagram is looped back  
This is a performance optimization for upper-layer protocols that restrict the membership of a group to one process per host (such as a routing protocol), or that handle loopback of group communication at a higher layer (such as a multicast transport protocol).  

### 2. Extensions to the IP Module
The logic of normal IP implementations :  
```
        if IP-destination is on the same local network:
           send datagram locally to IP-destination
        else:
           send datagram locally to GatewayTo( IP-destination )
```
The routing logic for allowing multicast transmissions:  
```
        if IP-destination is on the same local network or IP-destination is a host group:
           send datagram locally to IP-destination
        else:
           send datagram locally to GatewayTo( IP-destination )
```
### 3. Extensions to the Local Network Service Interface
No change to the local network service interface is required to support the sending of multicast IP datagrams.  
The IP module merely specifies an IP host group destination, rather than an individual IP destination.  
### 4. Extensions to an Ethernet Local Network Module
The Ethernet directly supports the sending of local multicast packets by allowing multicast addresses in the destination field of Ethernet packets.  All that is needed to support the sending of multicast IP datagrams is a procedure for **mapping IP host group addresses** to Ethernet multicast addresses.  

An IP host group address is mapped to an Ethernet multicast address by placing the low-order 23-bits of the IP address into the low-order 23 bits of the Ethernet multicast address 01-00-5E-00-00-00 (hex). Because there are 28 significant bits in an IP host group address, more than one host group address may map to the same Ethernet multicast address.  

### 5. Extensions to Local Network Modules other than Ethernet
All IP host group addresses might be mapped to the well-known local address of an IP multicast router; a router on such a network would take responsibility for completing multicast delivery within the network as well as among networks.  

## Rceiving Multicast IP datagrams

### 1. Extensions to the IP Service Interface

- IP service interface must be extended to provide two new operations:
    - JoinHostGroup  ( group-address, interface )
        - this host become a member of the host group identified by "group-address" on the given network interface
    - LeaveHostGroup ( group-address, interface )
        -  this host give up its membership in the host group identified by "group-address" on the given network interface
It is permissible to join the same group on more than one interface  

### 2. Extensions to the IP Module
為了支援 multicast IP datagram 接收，必須擴展 IP module 以維護與每個 network interface 關聯的主機組成員關係列表。  
 An incoming datagram with an IP host group address in its source address field is quietly discarded.  An ICMP error message (Destination Unreachable, Time Exceeded, Parameter Problem, Source Quench, or Redirect) is never generated in response to a datagram destined to an IP host group.  
The list of host group memberships is updated in response to JoinHostGroup and LeaveHostGroup requests from upper-layer protocols. Each membership should have an associated reference count or similar mechanism to handle multiple requests to join and leave the same group.  On the first request to join and the last request to leave a group on a given interface, the local network module for that interface is notified, so that it may update its multicast reception filter.  

IP module must also be extended  
IGMP is used to keep neighboring multicast routers informed of the host group memberships present on a particular local network.  
To support IGMP, every level 2 host must join the "all-hosts" group (address 224.0.0.1) on each network interface at initialization time and must remain a member for as long as the host is active.  

### 3. Extensions to the Local Network Service Interface
Incoming local network multicast packets are delivered to the IP module using the same "Receive Local" operation as local network unicast packets.  
the local network service interface is extended to provide two new operations:  
- JoinLocalGroup  ( group-address )
    - requests the local network module to accept and deliver up subsequently arriving packets destined to the given IP host group address
- LeaveLocalGroup ( group-address )
    - requests the local network module to stop delivering up packets destined to the given IP host group address.

 "group-address" is an IP host group address  
 The local network module is expected to map the IP host group addresses to local network addresses as required to update its multicast reception filter.  
Any local network module is free to ignore LeaveLocalGroup requests, and may deliver up packets destined to more addresses than just those specified in JoinLocalGroup requests, if it is unable to filter incoming packets adequately.  
  
   The local network module must not deliver up any multicast packets that were transmitted from that module; loopback of multicasts is handled at the IP layer or higher.  
### 4. Extensions to an Ethernet Local Network Module
### 5. Extensions to Local Network Modules other than Ethernet
all incoming broadcast packets can be accepted and passed to the IP module for IP-level filtering.  On point-to-point or store-and-forward networks, multicast IP datagrams will arrive as local network unicasts, so no change to the local network module should be necessary.  

## Internet Group Management Protocol (IGMP)
- is used by IP hosts to report their host group memberships to any immediately-neighboring multicast routers
-  an asymmetric protocol

### IGMP messages format
- IGMP messages of concern to hosts have the following format:
```

       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |Version| Type  |    Unused     |           Checksum            |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Group Address                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
- Type: There are two types of IGMP message of concern to hosts:
    - 1 = Host Membership Query
    - 2 = Host Membership Report
- Group Address
    - In a Host Membership Query message, the group address field is zeroed when sent, ignored when received.
    - In a Host Membership Report message, the group address field holds the IP host group address of the group being reported.

### Informal Protocol Description

### State Transition Diagram
- Non-Member state
- Delaying Member state
- Idle Member state
### 導致 IGMP 狀態轉換的五個 significant events
- **join group**
- **leave group**
- **query received**
- **report received**
- **timer expired**
### response to the significant events in 3 possible actions
- **send report**
- **start timer**
- **stop timer**

### diagram
```
                              ________________
                             |                |
                             |                |
                             |                |
                             |                |
                   --------->|   Non-Member   |<---------
                  |          |                |          |
                  |          |                |          |
                  |          |                |          |
                  |          |________________|          |
                  |                   |                  |
                  | leave group       | join group       | leave group
                  | (stop timer)      |(send report,     |
                  |                   | start timer)     |
          ________|________           |          ________|________
         |                 |<---------          |                 |
         |                 |                    |                 |
         |                 |<-------------------|                 |
         |                 |   query received   |                 |
         | Delaying Member |    (start timer)   |   Idle Member   |
         |                 |------------------->|                 |
         |                 |   report received  |                 |
         |                 |    (stop timer)    |                 |
         |_________________|------------------->|_________________|
                                timer expired
                                (send report)
```

## References
- https://ithelp.ithome.com.tw/articles/10067356
- https://zh.wikipedia.org/wiki/%E5%9B%A0%E7%89%B9%E7%BD%91%E7%BB%84%E7%AE%A1%E7%90%86%E5%8D%8F%E8%AE%AE
- https://en.wikipedia.org/wiki/IP_multicast
- https://www.cisco.com/c/en/us/td/docs/ios/solutions_docs/ip_multicast/White_papers/mcst_ovr.html#wp998638
- https://www.tldp.org/HOWTO/text/Multicast-HOWTO
- https://www.linuxjournal.com/article/6070
