# Log Managment

## Overview

The focus of this lab is about centrilised log managment. on particalure solution, an open soure one, is graylog.

### Prerequisites

One single requiremeent is needed is having dokcer installed in your systems. for step by step guide on docker instalion at [docker documentation](https://docs.docker.com/get-started/introduction/get-docker-desktop/)

### Task 1 - Starting Graylog

Start the docker-compose.yaml file to run graylog services.

```bash
docker compose -d
```

The previous command will tell docker to run the docker compose file. make sure to copy the content of .yaml as docker-compose.yaml

```yaml
services:
  mongodb:
    image: "mongo:6.0.18"
    ports:
      - "27017:27017"
    restart: "on-failure"
    networks:
      - graylog
    volumes:
      - "mongodb_data:/data/db"
      - "mongodb_config:/data/configdb"

  opensearch:
    image: "opensearchproject/opensearch:2.15.0"
    environment:
      - "OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g"
      - "bootstrap.memory_lock=true"
      - "discovery.type=single-node"
      - "action.auto_create_index=false"
      - "plugins.security.ssl.http.enabled=false"
      - "plugins.security.disabled=true"
      - "OPENSEARCH_INITIAL_ADMIN_PASSWORD=+_8r#wliY3Pv5-HMIf4qzXImYzZf-M=M"
    ulimits:
      memlock:
        hard: -1
        soft: -1
      nofile:
        soft: 65536
        hard: 65536
    ports:
      - "9203:9200"
      - "9303:9300"
    restart: "on-failure"
    networks:
      - graylog
    volumes:
      - "opensearch:/usr/share/opensearch/data"
  graylog:
    hostname: "server"
    image: "graylog/graylog-enterprise:6.1"
    depends_on:
      mongodb:
        condition: "service_started"
      opensearch:
        condition: "service_started"
    entrypoint: "/usr/bin/tini -- wait-for-it opensearch:9200 -- /docker-entrypoint.sh"
    environment:
      GRAYLOG_NODE_ID_FILE: "/usr/share/graylog/data/config/node-id"
      GRAYLOG_HTTP_BIND_ADDRESS: "0.0.0.0:9000"
      GRAYLOG_ELASTICSEARCH_HOSTS: "http://opensearch:9200"
      GRAYLOG_MONGODB_URI: "mongodb://mongodb:27017/graylog"
      GRAYLOG_REPORT_DISABLE_SANDBOX: "true"
      GRAYLOG_PASSWORD_SECRET: "somepasswordpepper"
      GRAYLOG_ROOT_PASSWORD_SHA2: "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"
      GRAYLOG_HTTP_EXTERNAL_URI: "http://127.0.0.1:9000/"
    ports:
      - "9000:9000/tcp"
      - "5044:5044/tcp"
      - "5140:5140/tcp"
      - "5140:5140/udp"
      - "12201:12201/tcp"
      - "12201:12201/udp"
      - "13301:13301/tcp"
      - "13302:13302/tcp"
    restart: "on-failure"
    networks:
      - graylog
    volumes:
      - "graylog_data:/usr/share/graylog/data"
networks:
  graylog:
    driver: "bridge"

volumes:
  mongodb_data:
  mongodb_config:
  opensearch:
  graylog_data:
```

Then go to http://127.0.0.1:9000 to open the graylog web interface.

Graylog servre need a way to reicve logs for diffrent endpoint solutions and appliences. In graylog terms, its inputs. By defauts multiple ports are open for diffrent logs sources (eg. syslog port 5140).

For this task we start by creating a custom input source. first, stop your gray log instance (docker compose down), then open a port (chosse a port number you like) in the graylog service (docker-compose.yaml), then start the services (docker compose up -d)

Second, create a new input at http://127.0.0.1:9000/system/inputs with input type == Raw/Plaintext TCP and with the same port number you open before.

Then try sending raw text to 127.0.0.1:<your_custom_port_number> (use nc command: echo "My log message" | nc 127.0.0.1 <your_custom_port_number>)

### Task 2 - Ingesting windows logs

Diffrent to linux syslog, Windows OS has diffrent log format for logging system & security events. Befor setting up, graylog log source, for windows os, we need to have sysmon to monitor windows activity & nx-log logger.

1. Sysmon : https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

   - configuration
     - `sysmon.exe -accepteula -i <config.file.xml>` (view ressource for config files)

2. NX-log: https://nxlog.co/downloads/nxlog-ce#nxlog-community-edition

   - config: C:\Program Files\nxlog\conf\nxlog.conf
     - change its content with config file (see ressource/nxlog.conf)

3. Create a new input in graylog UI
   - type = GELF UDP
   - leave every thing as default

Go to graylog dashboard to view all collected logs

**For linux users**
Instead, for linux endpoints, syslog is aleady supported, just edit /etc/rsyslog.conf with:

1. Enable TCP/UDP log forwareding
2. Edit remote ip to graylog ip (127.0.0.1)
