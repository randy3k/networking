# Prerequisite
```
sudo apt install isc-dhcp-client
```

# Bridge network namespaces to local network

First, setup the bridge and connect eth0 to it
```
DEVICE=eth0
sudo ip link add name br0 type bridge
sudo ip link set $DEVICE down
sudo ip address flush dev $DEVICE
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
sudo ip address show br0
# sudo ip route add default via 192.168.0.1 dev br0
```

To route traffic from the netns
```
sudo ip netns exec ns1 dhclient veth1b
sudo ip --netns ns1 address show veth1b
sudo ip netns exec ns1 ping 192.168.0.1
```

To show the devices associated with the bridge
```
sudo ip neigh show dev br0
```
