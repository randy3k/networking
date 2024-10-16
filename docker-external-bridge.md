# Bridge container directly to the local network

First, setup the bridge and connect eth0 to it
```bash
DEVICE=eth0
sudo ip link add name br0 type bridge
sudo ip link set $DEVICE down
sudo ip addr flush dev $DEVICE
sudo ip link set $DEVICE master br0

sudo ip link set br0 up
sudo ip link set $DEVICE up
```

To route the traffic from the bridge machine
```bash
sudo dhclient br0
```

We will run a container without any network. As it has no network intially, we need to make sure the image has all the tools required.
```bash
docker run -itd --name debian debian
docker exec debian bash -c 'apt-get update && apt-get install -y isc-dhcp-client iputils-ping'
docker commit debian debian_with_dhcp
docker rm -f debian
```

Connect a veth to the bridge
```bash
sudo ip link add veth0a type veth peer name veth0b
sudo ip link set veth0a master br0
sudo ip link set veth0a up
```

Connect a veth to the continaer and request IP from DHCP
```bash
docker run -itd --name n0 \
    --network none \
    --cap-add=NET_ADMIN \
    --mac-address=00:11:22:33:44:55 \
    debian_with_dhcp
PID=$(docker inspect -f '{{.State.Pid}}' n0)
sudo ip link set veth0b netns $PID
sudo ip link set veth0b up
docker exec -it n0 dhclient veth0b
```
