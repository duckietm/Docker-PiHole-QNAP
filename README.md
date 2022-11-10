# PiHole with Unboud for QNAP

Here is my setup of PiHole with the use of Unbound i hope it will be help full for all of you.

## My QNAP setup :

Lan 1 - Interface for daily traffic and has the default gateway

Lan 2 - Interface with only a static IP and only be used for Dockers

### Why this setup ?
As soon as i bind the interfaces it will not work as then an interface br0 will be made wich is not allowed to do Bridged with static IP's

### Why do you need this?
As we need port 53/tcp and 53/udp to be used, as this is already by used (dnsmask) for container service you are not able to use the port.

## Setup

I use as local LAN 192.168.0.0/24 so you need to change the subnet if you use any other.

from command line:
```cmd
cd /
mkdir docker
cd docker
mkdir pihole
```

then use your favorite text editior and make the following file in the pihole directory: docker-compose.yml
```yml
version: "3"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    networks:
      pihole:
        ipv4_address: 192.168.0.254
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "85:80/tcp"
    environment:
      TZ: 'Europe/Amsterdam'
      WEBPASSWORD: '### YOUR PASSWORD ###'
    dns:
      - 127.0.0.1
      - 1.1.1.1
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
    privileged: true

  unbound:
    container_name: unbound
    image: madnuttah/unbound:latest
    hostname: unbound
    networks:
      pihole:
        ipv4_address: 192.168.0.253
    ports:
      - 5335:5335/tcp
      - 5335:5335/udp
    environment:
      TZ: "Europe/Amsterdam"
    volumes:
      - ./unbound/unbound.conf:/usr/local/unbound/unbound.conf:rw #Your local path to Unbound
      - ./unbound/conf.d/:/usr/local/unbound/conf.d/:rw
      - ./unbound/log.d/:/usr/local/unbound/log.d/:rw
      - ./unbound/zones.d/:/usr/local/unbound/zones.d/:rw
    restart: unless-stopped

networks:
  pihole:
    driver: qnet
    driver_opts:
      iface: "eth0"
    ipam:
      driver: qnet
      options:
        iface: "eth0"
      config:
        - subnet: 192.168.0.0/24
          gateway: 192.168.0.1
```

No go to : https://github.com/duckietm/Docker-PiHole-QNAP and copy the compleet folder unbound into : /docker/pihole/ (i use WinSCP)
This has all the defaults for unbound ofcourse feel free to change it to your needs

You can now start the docker:
```cmd
cd /docker/pihole
docker-compose up -d
```

Open your browser and go to http://#your fixed pihole ip#/admin
Use your password (this was provided in the yml) and select settings in the right menu and select DNS
in the Custom 1 (IPv4) fill in the following:
IP of your unbound#5335
Expl. 192.168.0.253#5335

And thats it, you can test it and see what magic is happening !