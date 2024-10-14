# Create macvlan network (WIP)

Create a debian image with isc-dhcp-client
```
docker run -itd --name debian debian
docker exec debian bash -c 'apt-get update && apt-get install -y isc-dhcp-client'
docker commit debian randy3k/debian
docker rm -f debian
```

Create macvlan network
```
docker network create -d macvlan \
    --subnet=192.168.0.0/22 \
    --gateway=192.168.0.1 \
    -o parent=eth0 macvlan0
```

```
docker run -it \
    --network macvlan0 \
    --cap-add=NET_ADMIN \
    --mac-address=00:11:22:33:44:55 \
    --ip=192.168.0.68 \
    randy3k/debian 
```

```
ip addr show eth0
ip addr flush dev eth0
dhclient eth0
ip addr show eht0
```
