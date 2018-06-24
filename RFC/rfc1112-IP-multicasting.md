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

### 2. Extensions to the IP Module
### 3. Extensions to the Local Network Service Interface
### 4. Extensions to an Ethernet Local Network Module
### 5. Extensions to Local Network Modules other than Ethernet

## Internet Group Management Protocol (IGMP)
### IGMP messages format
### Informal Protocol Description
### State Transition Diagram
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

## References
- https://ithelp.ithome.com.tw/articles/10067356
- https://zh.wikipedia.org/wiki/%E5%9B%A0%E7%89%B9%E7%BD%91%E7%BB%84%E7%AE%A1%E7%90%86%E5%8D%8F%E8%AE%AE
- https://en.wikipedia.org/wiki/IP_multicast
- https://www.cisco.com/c/en/us/td/docs/ios/solutions_docs/ip_multicast/White_papers/mcst_ovr.html#wp998638
- https://www.tldp.org/HOWTO/text/Multicast-HOWTO
- https://www.linuxjournal.com/article/6070
