# VXLAN
> Virtual eXtensible Local Area Network

- Ref: https://tools.ietf.org/html/rfc7348

A Framework for Overlaying Virtualized Layer 2 Networks over Layer 3 Networks

## 為什麼會出現 VXLAN?
1. Limitations Imposed by Spanning Tree and VLAN Ranges
2. Multi-tenant Environments
3. Inadequate Table Sizes at ToR(Top-of-Rack) Switch

## Acronyms and Definitions
- PIM      Protocol Independent Multicast
- SPB      Shortest Path Bridging
- STP      Spanning Tree Protocol
- ToR      Top of Rack
- TRILL    Transparent Interconnection of Lots of Links
- VLAN     Virtual Local Area Network
- VNI      VXLAN Network Identifier (or VXLAN Segment ID)
- VTEP     VXLAN Tunnel End Point.  An entity that originates and/or
            terminates VXLAN tunnels
- VXLAN    Virtual eXtensible Local Area Network
- VXLAN Segment
            VXLAN Layer 2 overlay network over which VMs communicate
- VXLAN Gateway
            an entity that forwards traffic between VXLANs

## 什麼是 VXLAN
- VXLAN 全名 Virtual eXtensible Local Area Network。
- VXLAN is a Layer 2 overlay scheme on a Layer 3 network.
- Each overlay is termed a VXLAN segment.
- Only VMs within the same VXLAN segment can communicate with each other.
- Each VXLAN segment is identified through a 24-bit segment ID, termed the "VXLAN Network Identifier (VNI)".
- This allows up to 16 M VXLAN segments to coexist within the same administrative domain.

- VNI
    -  The VNI identifies the scope of the inner MAC frame originated by the individual VM.
        - 所以可能會有重複的 MAC address 但是不會有 traffic "cross over" 因為 the traffic is isolated using the VNI.
        - The VNI is in an outer header that encapsulates the inner MAC frame originated by the VM.

- VXLAN segment = VXLAN overlay network

- encapsulation: VXLAN could also be called a tunneling scheme to overlay Layer 2 networks on top of Layer 3 networks.
    - The tunnels are stateless
        - each frame is encapsulated according to a set of rules.

> 簡單解釋:
就是 VM 的封包原封不動，但在實體的 node 上做封裝 MAC address 的動作。實體主機的端點對端點溝通無礙，封包在實體主機中做封裝和解封裝動作，然後再傳給 VM。

### Unicast VM-to-VM Communication
當 VM1 在 VXLAN overlay 網路中，VM1 自己是不知道他是不是使用 VXLAN 網路的，這代表 VM1 不用做任何設定。  
當 VM1 要和另外一台 physical host 上的 VM2 溝通， VM1 會正常的送出一個 MAC frame 給 target 。  
physical host 的 VTEP 會查找與 VM1 的 VNI。physical host 會確認 MAC address 是否是同意網段，並且確認 destination 的 MAC address 和 remote VTEP 是可以 mapping 的起來。  
如果都確認 OK：
- 傳送:
    1. an outer header comprising an outer MAC, outer IP header, and VXLAN header are prepended to the original MAC frame.  
    2. The encapsulated packet is forwarded towards the remote VTEP. 
- 接收：
    - the remote VTEP verifies the validity of the VNI and whether or not there is a VM on that VNI using a MAC address that matches the inner destination MAC address.
        - 如果是
            1. 拆解封裝的 header 並傳遞到 VM2
The destination VM never knows about the VNI or that the frame was transported with a VXLAN encapsulation.  
- the remote VTEP learns the mapping from inner source MAC to outer source IP address.  It stores this mapping in a table so that when the destination VM sends a response packet, there is no need for an "unknown destination" flooding of the response packet.


### Broadcast Communication and Mapping to Multicast
-  With VXLAN, a header including the VXLAN VNI is inserted at the beginning of the packet along with the IP header and UDP header.
-  the broadcast packet is sent out to the IP multicast group on which that VXLAN overlay network is realized.
- have a mapping between the VXLAN VNI and the IP multicast group that it will use.
    - This mapping is done at the management layer and provided to the individual VTEPs through a management channel.
    - the VTEP can provide IGMP membership reports to the upstream switch/router to join/leave the VXLAN-related IP multicast groups as needed.
    - enable pruning of the leaf nodes for specific multicast traffic addresses based on whether a member is available on this host using the specific multicast address
    - multicast routing protocols like Protocol Independent Multicast - Sparse Mode (PIM-SM see [RFC4601]) will provide efficient multicast trees within the Layer 3 network.

- The destination VM sends a standard ARP response using IP unicast.
- This frame will be encapsulated back to the VTEP connecting the originating VM using IP unicast VXLAN encapsulation.

- Note that multicast frames and "unknown MAC destination" frames are also sent using the multicast tree, similar to the broadcast frames.

### Physical Infrastructure Requirements
- When IP multicast is used within the network infrastructure, a multicast routing protocol can be used by the individual Layer 3 IP routers/switches within the network.
- VTEPs MUST NOT fragment VXLAN packets
- Intermediate routers may fragment encapsulated VXLAN packets due to the larger frame size.
- The destination VTEP MAY silently discard such VXLAN fragments.
    -  To ensure end-to-end traffic delivery without fragmentation, it is RECOMMENDED that the MTUs (Maximum Transmission Units) across the physical network infrastructure be set to a value that accommodates the larger frame size due to the encapsulation.


## VXLAN Frame Format
### Figure 1: VXLAN Frame Format with IPv4 Outer Header
- an example of an inner Ethernet frame encapsulated within an outer Ethernet + IP + UDP + VXLAN header.  The outer destination MAC address in this frame may be the address of the target VTEP or of an intermediate Layer 3 router.  The outer VLAN tag is optional.  If present, it may be used for delineating VXLAN traffic on the LAN.  
  
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1

   Outer Ethernet Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |             Outer Destination MAC Address                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Outer Destination MAC Address | Outer Source MAC Address      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Outer Source MAC Address                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |OptnlEthtype = C-Tag 802.1Q    | Outer.VLAN Tag Information    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Ethertype = 0x0800            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Outer IPv4 Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |Protocl=17(UDP)|   Header Checksum             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Outer Source IPv4 Address               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                   Outer Destination IPv4 Address              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Outer UDP Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Source Port         |       Dest Port = VXLAN Port  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           UDP Length          |        UDP Checksum           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   VXLAN Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |R|R|R|R|I|R|R|R|            Reserved                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                VXLAN Network Identifier (VNI) |   Reserved    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Inner Ethernet Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |             Inner Destination MAC Address                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Inner Destination MAC Address | Inner Source MAC Address      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Inner Source MAC Address                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |OptnlEthtype = C-Tag 802.1Q    | Inner.VLAN Tag Information    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Payload:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Ethertype of Original Payload |                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
   |                                  Original Ethernet Payload    |
   |                                                               |
   |(Note that the original Ethernet Frame's FCS is not included)  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Frame Check Sequence:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   New FCS (Frame Check Sequence) for Outer Ethernet Frame     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

            Figure 1: VXLAN Frame Format with IPv4 Outer Header

###  Figure 2: VXLAN Frame Format with IPv6 Outer Header

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1

   Outer Ethernet Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |             Outer Destination MAC Address                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Outer Destination MAC Address | Outer Source MAC Address      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Outer Source MAC Address                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |OptnlEthtype = C-Tag 802.1Q    | Outer.VLAN Tag Information    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Ethertype = 0x86DD            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Outer IPv6 Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Traffic Class |           Flow Label                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Payload Length        | NxtHdr=17(UDP)|   Hop Limit   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                     Outer Source IPv6 Address                 +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                  Outer Destination IPv6 Address               +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Outer UDP Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Source Port         |       Dest Port = VXLAN Port  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           UDP Length          |        UDP Checksum           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   VXLAN Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |R|R|R|R|I|R|R|R|            Reserved                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                VXLAN Network Identifier (VNI) |   Reserved    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Inner Ethernet Header:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |             Inner Destination MAC Address                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Inner Destination MAC Address | Inner Source MAC Address      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                Inner Source MAC Address                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |OptnlEthtype = C-Tag 802.1Q    | Inner.VLAN Tag Information    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Payload:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Ethertype of Original Payload |                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
   |                                  Original Ethernet Payload    |
   |                                                               |
   |(Note that the original Ethernet Frame's FCS is not included)  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   Frame Check Sequence:
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   New FCS (Frame Check Sequence) for Outer Ethernet Frame     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

## VXLAN Deployment Scenarios
   +------------+-------------+
   |        Server 1          |
   | +----+----+  +----+----+ |
   | |VM1-1    |  |VM1-2    | |
   | |VNI 22   |  |VNI 34   | |
   | |         |  |         | |
   | +---------+  +---------+ |
   |                          |
   | +----+----+  +----+----+ |
   | |VM1-3    |  |VM1-4    | |
   | |VNI 74   |  |VNI 98   | |
   | |         |  |         | |
   | +---------+  +---------+ |
   | Hypervisor VTEP (IP1)    |
   +--------------------------+
                         |
                         |
                         |
                         |   +-------------+
                         |   |   Layer 3   |
                         |---|   Network   |
                             |             |
                             +-------------+
                                 |
                                 |
                                 +-----------+
                                             |
                                             |
                                      +------------+-------------+
                                      |        Server 2          |
                                      | +----+----+  +----+----+ |
                                      | |VM2-1    |  |VM2-2    | |
                                      | |VNI 34   |  |VNI 74   | |
                                      | |         |  |         | |
                                      | +---------+  +---------+ |
                                      |                          |
                                      | +----+----+  +----+----+ |
                                      | |VM2-3    |  |VM2-4    | |
                                      | |VNI 98   |  |VNI 22   | |
                                      | |         |  |         | |
                                      | +---------+  +---------+ |
                                      | Hypervisor VTEP (IP2)    |
                                      +--------------------------+

    Figure 3: VXLAN Deployment - VTEPs across a Layer 3 Network


## else
- Mac in UDP 的封裝協定
- Header 的 ID 欄位為 24 bit (2^24)

