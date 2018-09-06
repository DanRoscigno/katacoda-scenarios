### Deploy Elasticsearch and Kibana
`docker-compose -f /root/course/elasticsearch-kibana-compose.yml up -d`{{execute HOST1}}

### Check the health / readiness of Elasticsearch
`docker inspect elasticsearch | grep -A8 Health`{{execute HOST1}}

### Check the health / readiness of Kibana
`docker inspect kibana | grep -A8 Health`{{execute HOST1}}

### Start Filebeat
`docker run -d \
--net course_stack \
--name=filebeat \
--user=root \
--volume="/var/lib/docker/containers:/var/lib/docker/containers:rw" \
--volume="/root/course/filebeat.yml:/usr/share/filebeat/filebeat.yml:rw" \
--volume="/var/run/docker.sock:/var/run/docker.sock:rw" \
--detach=true docker.elastic.co/beats/filebeat:6.4.0 filebeat -e -strict.perms=false`{{execute HOST1}}

### Start NGINX
`docker run -d \
--net course_stack \
--label co.elastic.logs/module=nginx \
--label co.elastic.logs/fileset.stdout=access \
--label co.elastic.logs/fileset.stderr=error \
--name nginx \
-p 81:80 nginx`{{execute HOST1}}

Note in the NGINX run command there are three labels, these labels are available in the Docker environment, and Filebeat detects them and configures itself.  The three labels tell Filebeat that the module name **nginx** should be used to collect, parse, and visualize the logs from this container, and that the access logs are at STDOUT, while the error logs are at STDERR.
You can see these labels with the command:
`docker inspect nginx | grep -A4 Label`{{execute HOST1}}

