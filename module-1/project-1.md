# Connecting two network namespaces via veth

### Objectives
 - Create 2 network namespace
 - Connect them via veth
 - Establish and check connection

Assuming that, we are in root terminal

### Create 2 network namespace
```sh
ip netns add red
ip netns add green
```

### Create veth and connect to both namespaces
```sh
ip link add veth-red type veth peer name veth-green
ip link set veth-red netns red
ip link set veth-green netns green
```

### Assign ip address and up all interfaces
```sh
ip netns exec red ip addr add 192.168.1.10/24 dev veth-red
ip netns exec green ip addr add 192.168.1.20/24 dev veth-green
ip netns exec red ip link set lo up
ip netns exec red ip link set veth-red up
ip netns exec green ip link set lo up
ip netns exec green ip link set veth-green up
```

### Check connection
From red to green
```sh
ip netns exec red ping -c 3 192.168.1.20
```
Output
```sh
PING 192.168.1.20 (192.168.1.20) 56(84) bytes of data.
64 bytes from 192.168.1.20: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 192.168.1.20: icmp_seq=2 ttl=64 time=0.027 ms
64 bytes from 192.168.1.20: icmp_seq=3 ttl=64 time=0.026 ms

--- 192.168.1.20 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2033ms
rtt min/avg/max/mdev = 0.026/0.034/0.049/0.010 ms
```

From green to red
```sh
ip netns exec green ping -c 3 192.168.1.10
```

Output
```sh
PING 192.168.1.10 (192.168.1.10) 56(84) bytes of data.
64 bytes from 192.168.1.10: icmp_seq=1 ttl=64 time=0.014 ms
64 bytes from 192.168.1.10: icmp_seq=2 ttl=64 time=0.025 ms
64 bytes from 192.168.1.10: icmp_seq=3 ttl=64 time=0.027 ms

--- 192.168.1.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2042ms
rtt min/avg/max/mdev = 0.014/0.022/0.027/0.005 ms
```

### Cleanup
```sh
ip netns delete red
ip netns delete green
```


