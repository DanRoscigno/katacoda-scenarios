### Deploy the Elastic Beats

`kubectl create -f /root/course/filebeat-kubernetes.yaml `{{execute HOST1}}

`kubectl create -f /root/course/metricbeat-setup.yaml`{{execute HOST1}}

`kubectl create -f /root/course/metricbeat-kubernetes.yaml `{{execute HOST1}}

`kubectl create -f /root/course/packetbeat-kubernetes.yaml `{{execute HOST1}}

`kubectl get pods -n kube-system | grep beat`{{execute HOST1}}

