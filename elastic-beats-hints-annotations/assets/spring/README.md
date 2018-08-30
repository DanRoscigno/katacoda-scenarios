### Build the spring container:
 
`docker build -t gcr.io/${GOOGLE_CLOUD_PROJECT}/hello-java:v1 .`

### Test run the container
`docker run -ti --rm -p 8080:8080 gcr.io/${GOOGLE_CLOUD_PROJECT}/hello-java:v1`

### Push it to your private repo
`gcloud docker -- push gcr.io/${GOOGLE_CLOUD_PROJECT}/hello-java:v1`

