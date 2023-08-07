# se-test

## Prerequisted
installed minikube (v1.31.1)
installed kubectl(1.27.x)

## Patch existent apps
unpack zip archive to original_apps directory. it must be like:
```
# ls -1 original_apps
scraper_ops
scraper_worker
```

apply patches 
```
cd original_apps
patch -p1  < ../se-test/patches/scraper_ops.patch
patch -p1  < ../se-test/patches/scraper_worker.patch
```

## Build docker images
```
docker build --no-cache -t freezippo/ruby:3.1.0 -f Dockerfile.ruby-3.1.0 . && docker push freezippo/ruby:3.1.0
docker build --no-cache -t freezippo/rabbitmq:3.12.2 -f Dockerfile.rabbitmq . && docker push freezippo/rabbitmq:3.12.2
docker build --no-cache -t freezippo/scraper_ops:latest -f Dockerfile.scraper_ops ../../original_apps/scraper_ops && docker push freezippo/scraper_ops:latest
docker build --no-cache -t freezippo/scraper_worker:latest -f Dockerfile.scraper_worker ../../original_apps/scraper_worker && docker push freezippo/scraper_worker:latest
```

## apply k8s manifests
```
kubectl apply -f k8s/initial/ns.yaml
kubectl apply -n se-test -f configmap-scraper-ops.yaml

kubectl create secret docker-registry docker-hub -n se-test --docker-server=https://index.docker.io/v1/ --docker-username=freezippo --docker-password=xxxx --docker-email=xxx
kubectl create secret generic pg-db-data -n se-test --from-literal=password=se_pg_passw0rd --from-literal=username=postgres
kubectl create secret generic rabbitmq-data -n se-test --from-literal=password=myuser --from-literal=username=mypassw0rd

kubectl apply -n se-test -f k8s/postgresql/.
kubectl apply -n se-test -f k8s/rabbitmq/.
kubectl apply -n se-test -f k8s/redis/.

kubectl apply -n se-test -f k8s/scraper_ops/job.yaml

kubectl apply -n se-test -f k8s/scraper_ops/deployment-sidekiq.yaml k8s/scraper_ops/deployment-web.yaml k8s/scraper_ops/service-web.yaml
kubectl apply -n se-test -f k8s/scraper_workder/deployment.yaml 
```