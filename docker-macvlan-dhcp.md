# Create macvlan network and obtain IP from DHCP

Create a debian image with isc-dhcp-client
```
docker run -itd --name debian debian
docker exec debian bash -c 'apt-get update && apt-get install -y isc-dhcp-client'
docker commit debian randy3k/debian
docker rm -f debian
```

Create macvlan network without gateway and subnet. We will ignore the subnet assigned by docker.
```
docker network create -d macvlan -o parent=eth0 macvlan0
```

Run container with a fixed mac address.
```
docker run -it \
    --network macvlan0 \
    --cap-add=NET_ADMIN \
    --mac-address=00:11:22:33:44:55 \
    randy3k/debian 
```

```
ip addr show eth0
# remove the ip assigned by docker
ip addr flush dev eth0
dhclient eth0
ip addr show eht0
```
