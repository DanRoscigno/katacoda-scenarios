### Set the credentials
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

### Clone the heapster repo:

`git clone https://github.com/kubernetes/heapster.git /root/course/heapster`{{execute HOST1}}

### Deploy InfluxDB, Grafana, and heapster

`kubectl create -f /root/course/heapster/deploy/kube-config/influxdb/`{{execute HOST1}}

### Configure RBAC for heapster

`kubectl create -f /root/course/heapster/deploy/kube-config/rbac/heapster-rbac.yaml`{{execute HOST1}}

### Verify that heapster is available

You should see one running heapster pod:

`kubectl get pods -n kube-system | grep -E "monitoring|heapster"`{{execute HOST1}}

### Check to see if kube-state-metrics is running

`kubectl get pods --namespace=kube-system | grep kube-state`{{execute HOST1}}

and create it if needed (by default it will not be there)


`git clone https://github.com/kubernetes/kube-state-metrics.git kube-state-metrics`{{execute HOST1}}

`kubectl create -f kube-state-metrics/kubernetes`{{execute HOST1}}

`kubectl get pods --namespace=kube-system | grep kube-state `{{execute HOST1}}



### Deploy the Guestbook example
Note: This is mostly the default Guestbook example in the [Kubernetes.io docs](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/), and here is the [GitHub repo](https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml).

I added an ingress that preserves source IPs and added ConfigMaps for the Apache2 and Mod-Status configs so that I could block the /server-status endpoint from outside the internal network (actually apache2.conf is unedited, but I may need it later).  I also added a redis.conf to set the slowlog time criteria.


`kubectl create -f /root/course/guestbook.yaml `{{execute HOST1}}

Verify the external IP is assigned


`kubectl get service frontend -w`{{execute HOST1}}

Once the external IP address is assigned you can type CTRL-C to stop watching for changes and get the command prompt back (the -w is "watch for changes")

### Deploy the Elastic Beats

`kubectl create -f /root/course/filebeat-kubernetes.yaml `{{execute HOST1}}

`kubectl create -f /root/course/metricbeat-setup.yaml`{{execute HOST1}}

`kubectl create -f /root/course/metricbeat-kubernetes.yaml `{{execute HOST1}}

`kubectl create -f /root/course/packetbeat-kubernetes.yaml `{{execute HOST1}}

`kubectl get pods -n kube-system | grep beat`{{execute HOST1}}

### Deploy the Spring hello world app

`kubectl apply -f /root/course/hello-java.yaml `{{execute HOST1}}

### View in Kibana
The logs from the hellow world app will be available at this link:

These are being processed by the default Filebeat pattern, and are similar to what you would see with fluentd or other log shippers.  Next we will add annotations to the hello-java manifest to tell Filebeat how to stitch together the multi-line logs

### Update the hello-java metadata

`diff -U6 /root/course/hello-java.yaml /root/course/hello-java-hints.yaml`{{execute HOST1}}

`kubectl apply -f /root/course/hello-java-hints.yaml `{{execute HOST1}}

### Look at the changes in Kibana
At this point you will see that the multi-line pattern annotation has been applied and the logs look as you would like. Coming from Ops, I appreciate not having to have someone with a cluster admin role stop the agent, modify the config, and restart it.  Using these annotations puts the control of what gets collected in the hands of the application owner.  Just update the manifest and apply.

### View in Kibana

Open your Kibana URL and look under the Dashboard link, verify that the Apache and Redis dashboards are populating.

### Scale your deployments and see new pods being monitored
List the existing deployments:

`kubectl get deployments`{{execute HOST1}}

NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
frontend       3         3         3            3           3m
redis-master   1         1         1            1           3m
redis-slave    2         2         2            2           3m


Scale the frontend down to two pods:

`kubectl scale --replicas=2 deployment/frontend`{{execute HOST1}}

deployment "frontend" scaled


Check the frontend deployment:

`kubectl get deployment frontend`{{execute HOST1}}

NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
frontend   2         2         2            2           5m


### View the changes in Kibana
See the screenshot, add the indicated filters and then add the columns to the view.  You can see the ScalingReplicaSet entry that is marked, following from there to the top of the list of events shows the image being pulled, the volumes mounted, the pod starting, etc.
![Kibana Discover](https://raw.githubusercontent.com/elastic/examples/master/MonitoringKubernetes/scaling-discover.png)
