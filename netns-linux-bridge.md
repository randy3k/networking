# Prerequisite
```
sudo apt install -y isc-dhcp-client
sudo apt install -y iptables
```

# Bridge network namespaces to local network

First, setup the bridge and connect eth0 to it
```
DEVICE=eth0
sudo ip link add name br0 type bridge
sudo ip link set $DEVICE down
sudo ip addr flush dev $DEVICE
sudo ip link set $DEVICE master br0

sudo ip link set br0 up
sudo ip link set $DEVICE up
```

Then create network namespaces
```
sudo ip netns add ns1
sudo ip link add veth1a type veth peer name veth1b
sudo ip link set veth1a master br0
sudo ip link set veth1b netns ns1

sudo ip link set veth1a up
sudo ip --netns ns1 link set veth1b up
```

To route the traffic from the bridge machine
```
sudo dhclient br0
sudo ip addr show br0
# sudo ip route add default via 192.168.0.1 dev br0
```

To route traffic from the netns
```
sudo ip netns exec ns1 dhclient veth1b
sudo ip --netns ns1 addr show veth1b
sudo ip netns exec ns1 ping 192.168.0.1
```

To show the devices associated with the bridge
```
sudo ip neigh show dev br0
```

# Create a vlan and route traffic to/from them

```
sudo ip link add name br0 type bridge
sudo ip link set br0 up
```

Create the network namespaces
```
sudo ip netns add ns1
sudo ip link add veth1a type veth peer name veth1b
sudo ip link set veth1a netns ns1
sudo ip link set veth1b master br0

sudo ip --netns ns1 link set veth1a up
sudo ip link set veth1b up
```

```
sudo ip netns add ns2
sudo ip link add veth2a type veth peer name veth2b
sudo ip link set veth2a netns ns2
sudo ip link set veth2b master br0

sudo ip --netns ns2 link set veth2a up
sudo ip link set veth2b up
```

Set ip addresses to allow communication between `ns1` and `ns2`.
```
sudo ip --netns ns1 addr add 172.16.0.11/24 dev veth1a
sudo ip --netns ns2 addr add 172.16.0.12/24 dev veth2a
```

Set gateway to reach the local network / internet
```
sudo ip addr add 172.16.0.1/24 dev br0
sudo ip --netns ns1 route add default via 172.16.0.1
sudo ip --netns ns2 route add default via 172.16.0.1
```

Set NAT table
```
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sudo iptables --table nat -A POSTROUTING -s 172.16.0.1/24 -j MASQUERADE
```
