# Evolved Packet Core (EPC)

## Architecture

![](https://www.tutorialspoint.com/lte/images/lte_epc.jpg)

## components
#### HSS: Home Subscriber Server
- a central database: 包含有關所有網絡運營商訂戶的信息
- 從 UMTS 和 GSM 繼續發展

#### P-GW: Packet Data Network (PDN) Gateway
- 與外界溝通 ie: packet data networks PDN, using SGi interface.
- Each packet data network is identified by an access point name (APN)
- PDN gateway has the same role as the GPRS support node (GGSN) and the serving GPRS support node (SGSN) with UMTS and GSM

#### S-GW: serving gateway
- acts as a router
- forwards data between the base station and the PDN gateway

#### MME: mobility management entity
- 透過 means of signalling messages 和 Home Subscriber Server (HSS) 控制 high-level operation of the mobile

#### PCRF: Policy Control and Charging Rules Function
- 負責策略控制決策，以及控制駐留在 P-GW 中的策略控制執行功能（PCEF）中的 flow-based charging 功能。


serving 和 P-GW 的 interface 為 S5/S8。  
Ref:  
- https://www.tutorialspoint.com/lte/lte_network_architecture.htm  
- http://dasheyuan.com/post/lte-network-architecture/
