# Create ipvlan network

Suppose 192.168.0.1 is the router address. We also assume that IPs 192.168.0.4 - 192.168.0.7 (192.168.0.4/30) are not in use. Most of the following also applies to macvlan except that wireless NIC doesn't work well with macvlan.

```
DEVICE=eth0

docker network create -d ipvlan \
    --subnet=192.168.0.0/22 \
    --gateway=192.168.0.1 \
    --ip-range=192.168.0.4/30 \
    -o parent=$DEVICE ipvlan0

# test if the network is working
docker run --net=ipvlan0 --name=ipv1 --ip=192.168.0.4 -itd alpine /bin/sh 
docker run --net=ipvlan0 --name=ipv2 --ip=192.168.0.5 -it --rm alpine ping 192.168.0.4 -c 2
```

The containers and host [are](https://github.com/moby/moby/issues/21735#issuecomment-205904902) [disconnected](https://stackoverflow.com/questions/49600665/docker-macvlan-network-inside-container-is-not-reaching-to-its-own-host). A workaround is to create another ipvlan for the host.
```
sudo ip link add ipvl0 link $DEVICE type ipvlan mode l2
sudo ip link set ipvl0 up
sudo ip addr add 192.168.0.6/32 dev ipvl0
sudo ip route add 192.168.0.4/30 dev ipvl0
```

## Turn on promiscuous mode

We will need to turn on promiscuous mode in order for the NIC to accept all the arrived packets. It is particular importantant for macvlan. It might be also needed for ipvlan over wireless, see https://github.com/moby/moby/issues/43270
```
sudo ip link set $DEVICE promisc on
```

## To cleanup
```
docker stop ipv1
docker rm ipv1
docker network rm ipvlan0
sudo ip link delete ipvl0
```
