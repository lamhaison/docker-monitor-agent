# How you run docker agent
* Docker client to send metric to graphite backend. I have build docker agent to docker image. You can configure your own graphite server info.
* I use docker-compose to run docker agent on docker host and send metric to graphite (cpu, ram, total read, write disk and total tranfer, receiver bytes network)
* Docker-compose.yml file
```
* install docker-compose
pip install docker-compose

* Create docker-compose file
version: '2'
services:
  docker-monitor-agent_1:
    image: lamhaison/docker-agent:v1.12.3_interval
    restart: always
    network_mode: "host"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    container_name: "docker-agent_node1"
    environment:
      - GRAPHITE_SERVER=your_server_ip
      - GRAPHITE_PORT=2003
      - PREFIX=docker
      # interval 5s
      - INTERVAL=5000      
* Run docker agent
docker-compose -f docker-compose.yml up -d
```



# Graph on grafana
![alt tag](https://github.com/lamhaison/docker-monitor-agent/blob/master/Screen%20Shot%202017-09-17%20at%201.25.47%20AM.png)
![alt tag](https://github.com/lamhaison/docker-monitor-agent/blob/master/Screen%20Shot%202017-09-17%20at%201.27.24%20AM.png)



* Create docker-compose-graphite.yml to create graphite system
```
# docker-compose -f docker-compose-graphite.yml up -d
# Graphite, carbon, carbon-relay, grafana
version: '2'
services:
  carbon-relay:
    image: banno/carbon-relay
    volumes:
      - /docker/whisper:/opt/graphite/storage/whisper
    ports:
      - "2003:2003"
      - "2004:2004"
      - "7002:7002"
    container_name: "carbon-relay"
    depends_on:
      - carbon-cache_1
      - carbon-cache_2
    environment:
      - RELAY_METHOD=consistent-hashing
      - DESTINATIONS=carbon-cache_1:2004, carbon-cache_2:2004
  carbon-cache_1:
    image: banno/carbon-cache
    volumes:
      - /docker/whisper:/opt/graphite/storage/whisper
    container_name: "carbon-cache_1"

  carbon-cache_2:
    image: banno/carbon-cache
    volumes:
      - /docker/whisper:/opt/graphite/storage/whisper
    container_name: "carbon-cache_2"
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    container_name: "grafana"
    depends_on:
     - graphite
  graphite:
    image: banno/graphite-web
    volumes:
      - /docker/whisper:/opt/graphite/storage/whisper
    ports:
      - "80:80"
    container_name: "graphite"
```
