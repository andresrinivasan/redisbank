# Running on Kubernetes as a Deployment with Redis Enterprise

As a more generic Kubernetes strategy, a Kubernetes Deployment is used to create the pods for RedisBank. The RedisBank Docker image created by [Dockerfile](./Dockerfile) is pushed to a repository. The image is then referenced in the Deployment. Note the dependency on an already deployed Redis Enterprise database using the [Redis Enterprise Operator](https://docs.redis.com/latest/kubernetes/).

➡ This shows using GCP and GKE; the pattern will work anywhere

➡ It is left as an exercise to the reader to have a Kubernetes cluster and all the appropriate CLIs necessary

## Create a Docker image for a GKE Deployment

Create the Docker image and push it to your favorite repository. In this example we'll use Google Container Repository (GCR). First you'll need to add the OAuth credentials to Docker with 

```sh
gcloud auth configure-docker gcr.io
```

and then push the image with

```sh
docker build -t redisbank .
docker tag redisbank gcr.io/YOUR-GCP-PROJECT/redisbank
docker push gcr.io/YOUR-GCP-PROJECT/redisbank
```

## Set up a GKE Cluster

➡ It is left as an excercise to the reader to setup GKE.

## Add the Redis Enterprise Operator to the GKE cluster

Please see [Deploy Redis Enterprise Software on Kubernetes](https://docs.redis.com/latest/kubernetes/deployment/quick-start/)

## Create the Redis Enterprise cluster

A Redis Enterprise cluster is declared with the RedisEnterpriseCluster (REC) resource. Feel free to create the YAML yourself or use [this one](https://raw.githubusercontent.com/andresrinivasan/redis-enterprise-k8s-custom-resources/master/getting-started/rec.yaml) which is ready to go.

```sh
kubectl apply -f https://raw.githubusercontent.com/andresrinivasan/redis-enterprise-k8s-custom-resources/master/getting-started/rec.yaml
```

## Create the Redis Enterprise database

A Redis Enterprise database is declared with the RedisEnterpriseDatabase (REDB) resource. As part of this resource we also need to specify the modules (Search and Time Series in this case) needed by the RedisBank app. The supplied REDB creates a 1 GB single shard HA database.

```sh
kubectl apply -f https://raw.githubusercontent.com/andresrinivasan/redisbank/blob/work/kubernetes/redis-enterprise-database.yaml
```

## Start the RedisBank app

The provided deployment has everything you need EXCEPT the name of the Docker image for the app. Please update spec.image and replace `YOUR-IMAGE-URL` with the URL of your image. Then add the deployment to your Kubernetes cluster with 

```sh
kubectl apply -f kubernetes/redisbank-enterprise-deployment.yaml
```

## Troubleshooting
