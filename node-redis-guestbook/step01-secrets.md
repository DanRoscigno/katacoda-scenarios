### Network

Create a network shared between the app and the Elastic Stack:

`docker network create course_stack`{{execute HOST2}}

### Deploy Elasticsearch 

`
docker run -d \
  --name=elasticsearch \
  --env="discovery.type=single-node" \
  --env="ES_JAVA_OPTS=-Xms256m -Xmx256m" \
  --network=course_stack \
  -p 9300:9300 -p 9200:9200 \
  --health-cmd='curl -s -f http://localhost:9200/_cat/health' \
  docker.elastic.co/elasticsearch/elasticsearch:6.4.0 
`{{execute HOST2}}

### Check the health / readiness of Elasticsearch

In the run command that you just ran, there is a health check defined.  This connects to the cluster health API of Elasticsearch.  In the output of the following command you will see the test result.  Wait until it returns a healthy response before deploying Kibana.

`docker inspect elasticsearch | grep -A8 Health`{{execute HOST2}}

### Deploy Kibana

`
docker run -d \
  --name=kibana \
  --user=kibana \
  --network=course_stack -p 5601:5601 \
  --health-cmd='curl -s -f http://localhost:5601/login' \
  docker.elastic.co/kibana/kibana:6.4.0 
`{{execute HOST2}}

### Check the health / readiness of Kibana

In the run command that you just ran, there is a health check defined.  This connects to Kibana and ensures that it is available. In the output of the following command you will see the test result.  Wait until it returns a healthy response before deploying Beats, as the Beats need to connect to both Elasticsearch and Kibana to install the modules that customize the experience related to the apps you are running (NGINX, Apache HTTPD, etc.).

`docker inspect kibana | grep -A8 Health`{{execute HOST2}}

### Run the APM Server

`
docker run -d \
--name=apm-server \
--network=course_stack \
-p 8200:8200 \
docker.elastic.co/apm/apm-server:6.4.0
`{{execute HOST2}}

### Run Redis

`
docker run \
  --network=course_stack \
  --name redis \
  -d -p 6379:6379 redis
`{{execute HOST2}}

### Run the node-guestbook

`
docker run \
  --name guestbook \
  -p 8080:8080  \
  --network=course_stack \
  -d danroscigno/node-guestbook
`{{execute HOST2}}

### Generate some traffic through the Guestbook
At the top of the terminal you will see a Guestbook tab.  Click on that and do the right thing.

