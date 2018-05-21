# Using OVS bridge for docker networking
用 OVS 的 bridge 作為 docker network 的練習，以下都用 root 權限進行。

## 1. 安裝 OVS
```sh
# apt-get -y install openvswitch-switch
```

## 2. 安裝  ovs-docker utility
```sh
# cd /usr/bin
# wget https://raw.githubusercontent.com/openvswitch/ovs/master/utilities/ovs-docker
# chmod a+rwx ovs-docker
```

## 3. 新增一個 OVS bridge
```sh
# ovs-vsctl add-br ovs-br1
# ovs-vsctl show
7d4d0c73-3528-416f-bc1d-661ad1b95137
    Bridge "ovs-br1"
        Port "ovs-br1"
        Interface "ovs-br1"
        type: internal
    ovs_version: "2.5.4"
```

## 4. 在 ovs bridge 設定內網 IP
```sh
# ifconfig ovs-br1 192.168.0.1 netmask 255.255.0.0 up
```

## 5. 新增兩個 Docker container
以下兩個指令需要使用 tmux 用不同視窗來開。
```sh
# docker run -t -i --name container1 ubuntu /bin/bash
# docker run -t -i --name container2 ubuntu /bin/bash
```

## 6. 將 container 連接到 OVS bridge
再開一個 tmux 的視窗
```sh
# ovs-docker add-port ovs-br1 eth0 container1 --ipaddress=192.168.1.1/16 --gateway=192.168.0.1
# ovs-docker add-port ovs-br1 eth0 container2 --ipaddress=192.168.1.2/16 --gateway=192.168.0.1
```

## 7. 互 ping
a. 先在兩個 container 中下以下指令
```sh
# apt update && apt install iputils-ping net-tools
```

b. 在 container1 中來 ping 192.168.1.2
```sh
# ping 192.168.1.2
```
c. 在 container2 中來 ping 192.168.1.1
```sh
# ping 192.168.1.1
```

## 8. 加入 NAT rules 讓 container 可以 ping 外面
(我的對外 network interface 是 `enp0s3`)
```sh
root@docker:~# export pubintf=enp0s3
root@docker:~# export privateintf=ovs-br1
root@docker:~# iptables -t nat -A POSTROUTING -o $pubintf -j MASQUERADE
root@docker:~# iptables -A FORWARD -i $privateintf -j ACCEPT
root@docker:~# iptables -A FORWARD -i $privateintf -o $pubintf -m state --state RELATED,ESTABLISHED -j ACCEPT
```
這樣兩個 container 就可以 `ping 8.8.8.8` 了

Ref:
- http://containertutorials.com/network/ovs_docker.html  
- https://developer.ibm.com/recipes/tutorials/using-ovs-bridge-for-docker-networking/  
