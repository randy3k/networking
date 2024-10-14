# Isolate cloudflared from my LAN network

```
# docker-compose.yaml
networks:
  cloudflared-network:
    name: cloudflared-network
    driver: bridge
    ipam:
     config:
       - subnet: 10.0.1.0/24
         gateway: 10.0.1.1

services:
  nginx-reverse-proxy:
    image: nginx:latest
    restart: unless-stopped
    container_name: nginx-reverse-proxy
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      cloudflared-network:
        ipv4_address: 10.0.1.3

  cloudflared:
    image: cloudflare/cloudflared
    container_name: cloudflared
    restart: unless-stopped
    command: ["tunnel", "--no-autoupdate", "run"]
    environment:
      - TUNNEL_TOKEN=${TUNNEL_TOKEN}
    networks:
      cloudflared-network:
        ipv4_address: 10.0.1.2
    depends_on:
      - nginx-reverse-proxy

  iptables:
    build:
      dockerfile_inline: |
        FROM alpine:latest
        # assume that the host is using iptables-nft, not iptables-legacy
        RUN apk update & apk add iptables
    container_name: iptables
    restart: unless-stopped    
    cap_add:
      - NET_ADMIN
    network_mode: host
    configs:
      - source: block.sh
        target: /block.sh
      - source: unblock.sh
        target: /unblock.sh
    environment:
      - CLOUDFLARED_IP=10.0.1.2
      - LAN_SUBNET=192.168.0.0/16
    command: /bin/sh /block.sh
    depends_on:
      - cloudflared

# inline file requires docker 2.23.1
configs:
  block.sh:
      content: |
        #! /bin/sh
        trap "/bin/sh /unblock.sh" INT TERM
        iptables -C DOCKER-USER -d $$LAN_SUBNET -s $$CLOUDFLARED_IP -j REJECT 2>/dev/null || \
          iptables -I DOCKER-USER -d $$LAN_SUBNET -s $$CLOUDFLARED_IP -j REJECT
        iptables -C INPUT -s $$CLOUDFLARED_IP -j REJECT 2>/dev/null || \
          iptables -I INPUT -s $$CLOUDFLARED_IP -j REJECT
        sleep infinity &
        wait
  unblock.sh:
      content: |
        #! /bin/sh
        iptables -C DOCKER-USER -d $$LAN_SUBNET -s $$CLOUDFLARED_IP -j REJECT 2>/dev/null &&\
          iptables -D DOCKER-USER -d $$LAN_SUBNET -s $$CLOUDFLARED_IP -j REJECT
        iptables -C INPUT -s $$CLOUDFLARED_IP -j REJECT 2>/dev/null && \
          iptables -D INPUT -s $$CLOUDFLARED_IP -j REJECT
```