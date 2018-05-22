# OpenvSwitch Architecure

## dpif providers fit into the Open vSwitch architecture
```sh
           _
          |   +-------------------+
          |   |    ovs-vswitchd   |<-->ovsdb-server
          |   +-------------------+
          |   |      ofproto      |<-->OpenFlow controllers
          |   +--------+-+--------+  _
          |   | netdev | |ofproto-|   |
userspace |   +--------+ |  dpif  |   |
          |   | netdev | +--------+   |
          |   |provider| |  dpif  |   |
          |   +---||---+ +--------+   |
          |       ||     |  dpif  |   | implementation of
          |       ||     |provider|   | ofproto provider
          |_      ||     +---||---+   |
                  ||         ||       |
           _  +---||-----+---||---+   |
          |   |          |datapath|   |
   kernel |   |          +--------+  _|
          |   |                   |
          |_  +--------||---------+
                       ||
                    physical
                       NIC
```

### datapath
在 kernel 層，負責 data 的交換


Ref:  
- https://feisky.gitbooks.io/sdn/ovs/internal.html
- http://docs.openvswitch.org/en/latest/topics/porting/
- http://docs.openvswitch.org/en/latest/topics/porting/#open-vswitch-architectural-overview
- https://www.jianshu.com/p/bf112793d658
- http://www.cnblogs.com/popsuper1982/p/5848879.html
- https://software.intel.com/en-us/articles/ovs-dpdk-datapath-classifier
- https://mlwmlw.org/2012/04/open-vswitch-component-overview/
