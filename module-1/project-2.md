# Bridge network among namespaces

### Objectives
 - Create a bridge and 2 network namespace
 - Connect bridge to host and 2 namespaces to bridge
 - Establish and check connection

Assuming that, we are in root terminal

### Install required packages
```sh
apt update && apt -y upgrade && apt install -y iproute2 iputils-ping net-tools
```

### Create bridge and 2 network namespace
```sh
ip link add v-net type bridge
ip netns add red
ip netns add green
```

### Create veth and connect to both namespaces
```sh
ip link add veth-red-ns type veth peer name veth-red-br
ip link add veth-green-ns type veth peer name veth-green-br
ip link set veth-red-ns netns red
ip link set veth-green-ns netns green
ip link set veth-red-br master v-net
ip link set veth-green-br master v-net
```

### Assign ip address and up all interfaces
```sh
ip addr add 10.0.0.1/24 dev v-net
ip netns exec red ip addr add 10.0.0.10/24 dev veth-red-ns
ip netns exec green ip addr add 10.0.0.20/24 dev veth-green-ns
ip link set v-net up
ip netns exec red ip link set lo up
ip netns exec red ip link set veth-red-ns up
ip link set veth-red-br up
ip netns exec green ip link set lo up
ip netns exec green ip link set veth-green-ns up
ip link set veth-green-br up
```


### Add route entry
```sh
ip netns exec red ip route add default via 10.0.0.1
ip netns exec green ip route add default via 10.0.0.1
```

### Check connection
From red to green
```sh
ip netns exec red ping -c 3 10.0.0.20
```
Output
```sh
PING 10.0.0.20 (10.0.0.20) 56(84) bytes of data.
64 bytes from 10.0.0.20: icmp_seq=1 ttl=64 time=0.071 ms
64 bytes from 10.0.0.20: icmp_seq=2 ttl=64 time=0.043 ms
64 bytes from 10.0.0.20: icmp_seq=3 ttl=64 time=0.044 ms
--- 10.0.0.20 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2038ms
rtt min/avg/max/mdev = 0.043/0.052/0.071/0.013 ms
```

From green to red
```sh
ip netns exec green ping -c 3 10.0.0.10
```

Output
```sh
PING 10.0.0.10 (10.0.0.10) 56(84) bytes of data.
64 bytes from 10.0.0.10: icmp_seq=1 ttl=64 time=0.048 ms
64 bytes from 10.0.0.10: icmp_seq=2 ttl=64 time=0.049 ms
64 bytes from 10.0.0.10: icmp_seq=3 ttl=64 time=0.044 ms
--- 10.0.0.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2045ms
rtt min/avg/max/mdev = 0.044/0.047/0.049/0.002 ms
```

### Cleanup
```sh
ip netns delete red
ip netns delete green
ip link delete v-net type bridge
```

