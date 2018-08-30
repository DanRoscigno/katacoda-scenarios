### Deploy the Spring hello world app

`kubectl apply -f /root/course/hello-java.yaml `{{execute HOST1}}

### View in Kibana
The logs from the hellow world app will be available at this link:

These are being processed by the default Filebeat pattern, and are similar to what you would see with fluentd or other log shippers.  Next we will add annotations to the hello-java manifest to tell Filebeat how to stitch together the multi-line logs

