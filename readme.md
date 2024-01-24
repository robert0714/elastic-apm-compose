# User docker compose to stand up whole stack Elasticsearch, Kibana and APM server
You can ^C to stop afterwards you can use `start` and `stop` commands
THIS IS NOT A PRODUCTION SETUP 

Docker compose file elastic-apm-compose.yml found here : https://gist.github.com/bvader/9665fa7b3bd69457517e41a7c28b4725

```bash 
TAG=7.17.14 docker-compose -f elastic-apm-compose.yml up -d

TAG=7.10.2 docker-compose -f elastic-apm-compose-oss.yml  up -d
```

## After the stack is full running please go to
http://localhost:5601/app/kibana#/home/tutorial/apm?_g=()

# And check the following
  The Server Status and  
  The Agent will say it is already good that is a minor bug
 Load Kibana Objects
 Now you are ready to go.


## Clone Petclinic Repo Java Spring Boot App

``
git clone https://github.com/bvader/spring-petclinic.git

cd spring-petclinic

# Build
./mvnw package -Dmaven.test.skip=true

# Get the Elastic Java APM agent
curl -O  https://search.maven.org/remotecontent?filepath=co/elastic/apm/elastic-apm-agent/1.28.4/elastic-apm-agent-1.28.4.jar

curl -O  https://search.maven.org/remotecontent?filepath=co/elastic/apm/apm-agent-attach-cli/1.28.4/apm-agent-attach-cli-1.28.4.jar


# Run Petclinc with APM, 
NOTE: this enables method tracing 

## Unix / Mac OS
```bash
java -javaagent:/home/vagrant/elastic-apm-agent-1.28.4.jar \
-Delastic.apm.server_urls="http://10.100.98.200:8200" \
-Delastic.apm.service_name="spring-petclinic-monolith" \
-Delastic.apm.application_packages="org.springframework.samples" \
-Delastic.apm.trace_methods="org.springframework.samples.petclinic.*" \
-jar  spring-petclinic/target/spring-petclinic-2.1.0.BUILD-SNAPSHOT.jar
```

java -jar  apm-agent-attach-cli-1.28.4.jar \
    --exclude-user root \
    --include-main MyApplication spring-petclinic/target/spring-petclinic-2.1.0.BUILD-SNAPSHOT.jar \
    --include-vmargs elastic.apm.agent.attach=true \
    --continuous \
    --config service_name=m"spring-petclinic-monolith" \
    --config server_url=http://localhost:8200

## Windows
```bash
java -javaagent:./elastic-apm-agent-1.28.4.jar `
"-Delastic.apm.server_urls=http://localhost:8200" `
"-Delastic.apm.service_name=spring-petclinic-monolith" `
"-Delastic.apm.application_packages=org.springframework.samples" `
"-Delastic.apm.trace_methods=org.springframework.samples.petclinic.*" `
-jar target/spring-petclinic-2.1.0.BUILD-SNAPSHOT.jar
```


# Point your browser to http://localhost:8200
# Click around in the petclince app and navigate to APM in Kibana
# You should see transactions front end and back end. 
# In the Petclinic App got to Find Owners
# 1) Search with Last Name : <empty>, returns all Ownders Fast
# 2) Search with Last Name : Davis, returns 2 Owners Fast
# 3) Search with Last Name : Deckard, Returns no Owners...very slow
# Can you figure out why? Where is the bad method?

# Then point your own apps if you want

# User docker compose to stand up whole stack Elasticsearch, Kibana and APM server
TAG=7.17.14 docker-compose -f elastic-apm-compose.yml up
TAG=7.17.14 docker-compose -f elastic-apm-compose.yml down

TAG=7.17.14 docker-compose -f elastic-apm-compose.yml start
TAG=7.17.14 docker-compose -f elastic-apm-compose.yml stop
# IF You need prefic apm
```yaml
version: '3.1'
services:
  elasticsearch:
    container_name: elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17
    environment: ['ES_JAVA_OPTS=-Xms2g -Xmx2g','bootstrap.memory_lock=true','discovery.type=single-node', 'http.host=0.0.0.0', 'transport.host=127.0.0.1']
    restart: always
    ports:
    - "9200:9200"
    networks: ['stack']
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:7.17
    ports:
    - "5601:5601"
    environment: ['SERVER_BASEPATH=/apm','SERVER_REWRITEBASEPATH=true']
    restart: always
    networks: ['stack']
    depends_on: ['elasticsearch']

  apm-server:
    container_name: apm
    restart: always
    image: docker.elastic.co/apm/apm-server:7.17
    ports:
    - "8200:8200"
    - "6060:6060"
    networks: ['stack']
    depends_on: ['elasticsearch']
    command: >
      apm-server -e
        -E apm-server.rum.enabled=true
        -E apm-server.kibana.path=/apm
        -E apm-server.rum.event_rate.limit=1000
networks: {stack: {}}
```
# Official Document
* https://www.elastic.co/guide/en/observability/current/apm.html
* 
