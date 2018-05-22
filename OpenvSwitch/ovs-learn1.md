# 初步認識 OpenvSwitch
- a multilayer software switch
- supports standard management interfaces and opens the forwarding functions to programmatic extension and control.
- support distribution across multiple physical servers.
-  supports the features:
   - Standard 802.1Q VLAN model with trunk and access ports
   - NIC bonding with or without LACP on upstream switch
   - NetFlow, sFlow(R), and mirroring for increased visibility
   - QoS (Quality of Service) configuration, plus policing
   - Geneve, GRE, VXLAN, STT, and LISP tunneling
   - 802.1ag connectivity fault management
   - OpenFlow 1.0 plus numerous extensions
   - Transactional configuration database with C and Python bindings
   - High-performance forwarding using a Linux kernel module

- Linux kernel module supports Linux 3.10

## components

### ovs-vswitchd
a daemon that implements the switch, along with a companion Linux kernel module for flow-based switching.
### ovsdb-server
a lightweight database server that ovs-vswitchd queries to obtain its configuration.
### ovs-dpctl
a tool for configuring the switch kernel module.
### ovs-vsctl
a utility for querying and updating the configuration of ovs-vswitchd.
### ovs-appctl
a utility that sends commands to running Open vSwitch daemons.
### Scripts and specs
for building RPMs for Citrix XenServer and Red Hat Enterprise Linux. The XenServer RPMs allow Open vSwitch to be installed on a Citrix XenServer host as a drop-in replacement for its switch, with additional functionality.
### tools
- ovs-ofctl, a utility for querying and controlling OpenFlow switches and controllers.
- ovs-pki, a utility for creating and managing the public-key infrastructure for OpenFlow switches.
- ovs-testcontroller, a simple OpenFlow controller that may be useful for testing (though not for production).
- A patch to tcpdump that enables it to parse OpenFlow messages.

