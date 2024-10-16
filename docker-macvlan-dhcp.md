# Create macvlan network and obtain IP from DHCP

If you setup the macvlan network without any subnet, docker will automatically assign one to it. As we are going to obtain an IP from DHCP, we actually don't care what subnet it is.

```bash
docker network create -d macvlan -o parent=eth0 macvlan0
```

Since docker is going to assign a non exisiting subnet to the container, the container will have no internet access initially. You will need to make sure that the container has the dhcp client.

For debian, a image with dhcp could be created by
```bash
docker run -itd --name debian debian
docker exec debian bash -c 'apt-get update && apt-get install -y isc-dhcp-client'
docker commit debian debian_with_dhcp
docker rm -f debian
```

Then we need to flush the current assigned IP and obtain one from DHCP server via eth0.
It could be done by `ip addr flush dev eth0 && dhclient eth0`  before running the entrypoint of your image.

```bash
docker run -it --rm \
    --network macvlan0 \
    --cap-add=NET_ADMIN \
    --mac-address=00:11:22:33:44:55 \
    --entrypoint /bin/bash \
    debian_with_dhcp \
    -c "ip addr flush dev eth0 && dhclient eth0 && /bin/bash"
```
