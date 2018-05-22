# netlink
> communication between kernel and user space

## program
```c
       #include <asm/types.h>
       #include <sys/socket.h>
       #include <linux/netlink.h>

       netlink_socket = socket(AF_NETLINK, socket_type, netlink_family);
```

### description


- Ref: http://man7.org/linux/man-pages/man7/netlink.7.html
