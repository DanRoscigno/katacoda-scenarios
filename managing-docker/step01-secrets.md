### Set the credentials
`docker run -d \
--name=filebeat \
--user=root \
--volume="/var/lib/docker/containers:/var/lib/docker/containers:rw" \
--volume="/root/course/filebeat.yml:/usr/share/filebeat/filebeat.yml:rw" \
--volume="/var/run/docker.sock:/var/run/docker.sock:rw" \
--detach=true docker.elastic.co/beats/filebeat:6.4.0 filebeat -e -strict.perms=false`{{execute HOST1}}

`docker run -d --label co.elastic.logs/module=nginx --label co.elastic.logs/fileset.stdout=access --label co.elastic.logs/fileset.stderr=error -d -p 81:80 nginx`{{execute HOST1}}

`docker-compose -f /root/course/elasticsearch-kibana-compose.yml up -d`{{execute HOST1}}


Set these with the values from the http://cloud.elastic.co deployment:

```
vi ELASTIC_PASSWORD
vi ELASTIC_CLOUD_ID
```

and create a secret in the Kubernetes system level namespace:


`cd /root/course && kubectl create secret generic dynamic-logging \
--from-file=./ELASTIC_PASSWORD --from-file=./ELASTIC_CLOUD_ID \
--namespace=kube-system`{{execute HOST1}}


heapster is a container provided by Kubernetes to make metrics about Nodes, pods, etc. available.  If you ever ran `kubectl top pods` you have interacted with heapster. Metricbeat polls heapster for metrics and provides them to Elasticsearch. heapster can use various backends, we will deploy InfluxDB here.

