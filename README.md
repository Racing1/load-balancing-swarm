Load Balancing Swarm
===

This repository contains the files for the docker image, that will create the load balancing configuration for nginx using consul template. It requires consul as the key value store. It works with docker swarm and multi-host networking.

Usage
---

Set the following environment variables:

* `APP_NAME`: The container service name that we want to load balance.
* `CONSUL_URL`: URL endpoint for consul instance.

Docker Compose
---

Sample docker-compose.yml configuration is given below:

```
version: '2'

services:
  lb:
    image: hanzel/load-balancing-swarm
    container_name: lb
    ports:
      - "80:80"
    environment:
      - APP_NAME=tutum-nodejs-redis
      - CONSUL_URL=${KV_IP}:8500
    depends_on:
      - web
    networks:
      - front-tier

  web:
    image: hanzel/tutum-nodejs-redis
    ports:
      - "4000"
    environment:
      - APP_PORT=4000
      - REDIS_IP=redis
      - REDIS_PORT=6379
    depends_on:
      - redis
    networks:
      - front-tier
      - back-tier

  redis:
    image: redis
    container_name: redis
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - back-tier

volumes:
  redis-data:
    driver: local

networks:
  front-tier:
    driver: overlay
  back-tier:
    driver: overlay
```