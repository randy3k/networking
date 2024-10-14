# Allow internal bridged network to gain internet access

```
docker network create net0 \
    --internal \
    --subnet=172.18.0.0/16 \
    --gateway=172.18.0.1 \
    -o com.docker.network.bridge.name=br0

docker run --net=net0 --name=n0 --cap-add=NET_ADMIN -itd alpine /bin/sh 
docker run --net=net0 --name=n1 -it --rm alpine ping n0 -c 2

docker exec n0 ip route add default via 172.18.0.1
docker exec n0 ip route add 192.168.0.0/24 via 172.18.0.1
sudo iptables --table nat -A POSTROUTING -s 172.18.0.1/16 -j MASQUERADE
# to override the rules in DOCKER-ISOLATION-STAGE-1
sudo iptables -I DOCKER-USER -d 172.18.0.0/16 -j ACCEPT
sudo iptables -I DOCKER-USER -s 172.18.0.0/16 -j ACCEPT
docker exec n0 ping www.google.com -c 2
```

To clean up
```
docker stop n0
docker rm n0
docker network rm net0
sudo iptables --table nat -D POSTROUTING -s 172.18.0.1/16 -j MASQUERADE
sudo iptables -D DOCKER-USER -d 172.18.0.0/16 -j ACCEPT
sudo iptables -D DOCKER-USER -s 172.18.0.0/16 -j ACCEPT
```