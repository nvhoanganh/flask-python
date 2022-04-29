# See Code Level Metrics inside you IDE with CodeStream and NewRelic

## Step 1: deploy a plain Flask app to k8s

```bash
# build the docker image
docker buildx build . --platform linux/amd64 -t anthonynguyen334/flask-codestream --progress=plain

# push the docker image
docker tag anthonynguyen334/flask-codestream anthonynguyen334/flask-codestream:latest
docker push anthonynguyen334/flask-codestream:latest

# deploy to your k8s cluster
kubectl apply -f k8s.yaml -n sock-shop

# get public IP address of the service
kubectl get service --watch --namespace=sock-shop

# make sure you connect to it
curl http://<YOURPUBLICIP>/error

# install https://github.com/rakyll/hey
brew install hey

# load test the Flask API
hey -n 2000 http://20.121.251.151/ping
```

## Step 2: Add newrelic apm Agent

```bash
# build newrelic base image
docker buildx build -f NewRelicBaseImageDockerFile . --platform linux/amd64 -t python_newrelic:latest --progress=plain


# update Dockerfile and replace 'FROM python:3.8-slim-buster' to 'FROM python_newrelic:latest' and build image again
docker buildx build . --platform linux/amd64 -t anthonynguyen334/flask-codestream:withNRApm --progress=plain
# push the updated image
docker push anthonynguyen334/flask-codestream:withNRApm

# update the image of the deployment
kubectl set image deployment/flask-simple \
    flask-simple=anthonynguyen334/flask-codestream:withNRApm \
    -n sock-shop

# Set required env variables
kubectl set env deployment/flask-simple \
    NEW_RELIC_METADATA_REPOSITORY_URL=https://github.com/nvhoanganh/front-end.git \
    NEW_RELIC_METADATA_COMMIT=68c70ed16c951f73b3380c928deca2a8b3888698 \
    --namespace=sock-shop
```
