# Create ipvlan network

Suppose 192.168.0.1 is the router address via device wlan0. We also assume that 192.168.0.3, 192.168.0.4 - 192.168.0.7 (192.168.0.4/30) are not in use.

```
docker network create -d ipvlan \
    --subnet=192.168.0.0/22 \
    --gateway=192.168.0.1 \
    --ip-range=192.168.0.4/30 \
    -o parent=wlan0 ipvlan0

# test if the network is working
docker run --net=ipvlan0 --name=ipv1 --ip=192.168.0.4 -itd alpine /bin/sh 
docker run --net=ipvlan0 --name=ipv2 --ip=192.168.0.5 -it --rm alpine ping 192.168.0.4 -c 2
```

The containers and host are [disconnected](https://stackoverflow.com/questions/49600665/docker-macvlan-network-inside-container-is-not-reaching-to-its-own-host), create a network namespace on host to allow communication.

```
sudo ip link add ipvl0 link wlan0 type ipvlan mode l2
sudo ip link set ipvl0 up
sudo ip addr add 192.168.0.3/32 dev ipvl0
sudo ip route add 192.168.0.4/30 dev ipvl0
```

Allow wlan0 to receive packet for the containers, see https://github.com/moby/moby/issues/43270
```
sudo ip link set wlan0 promisc on
```

To cleanup
```
docker stop ipv1
docker rm ipv1
docker network rm ipvlan0
sudo ip link delete ipvl0
```
