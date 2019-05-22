Blah

`kubectl apply -f https://download.elastic.co/downloads/eck/0.8.0/all-in-one.yaml`{{execute HOST1}}

`kubectl apply -f /root/course/es_kb.yaml`{{execute HOST1}}

`PASSWORD=$(kubectl get secret quickstart-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode)`{{execute HOST1}}

`echo $PASSWORD`{{execute HOST1}}

- [Kibana](https://[[HOST_SUBDOMAIN]]-30601-[[KATACODA_HOST]].environments.katacoda.com/app/kibana)
