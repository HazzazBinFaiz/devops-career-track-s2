# Establish Egress traffic

### Objectives
 - Create a bridge and a network namespace
 - Connect namespace to bridge
 - Establish egress traffic and check connection

Assuming that, we are in root terminal

### Install required packages
```sh
apt update && apt -y upgrade && apt install -y iproute2 iputils-ping net-tools tcpdump iptables
```

### Create bridge and a network namespace
```sh
ip link add v-net type bridge
ip netns add red
```

### Create veth and connect to a namespaces
```sh
ip link add veth-red-ns type veth peer name veth-red-br
ip link set veth-red-ns netns red
ip link set veth-red-br master v-net
```

### Assign ip address and up all interfaces
```sh
ip addr add 192.168.0.1/24 dev v-net
ip netns exec red ip addr add 192.168.0.10/24 dev veth-red-ns
```

### Up all interfaces and bridge
```sh
ip link set v-net up
ip netns exec red ip link set lo up
ip netns exec red ip link set veth-red-ns up
ip link set veth-red-br up
```

### Add route and SNAT rule
```sh
ip netns exec red ip route add default via 192.168.0.1
iptables -t nat -A POSTROUTING -s 192.168.0.0/24  -j MASQUERADE
```

### Check connection
From red to internet
```sh
ip netns exec red ping -c 3 8.8.8.8
```

Output
```sh
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=1.00 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=0.979 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=0.998 ms
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.979/0.993/1.004/0.010 ms
```

### Cleanup
```sh
ip netns delete red
ip link delete v-net type bridge
```

