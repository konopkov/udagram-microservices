# Udagram Image Filtering Microservice

Udagram is a simple cloud application developed alongside the Udacity Cloud Engineering Nanodegree. It allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice.

The project is split into three parts:
1. [The Simple Frontend](/udacity-c3-frontend)
A basic Ionic client web application which consumes the RestAPI Backend.
2. [The RestAPI Feed Backend](/udacity-c3-restapi-feed), a Node-Express feed microservice.
3. [The RestAPI User Backend](/udacity-c3-restapi-user), a Node-Express user microservice.

## Deployment

### Locally

Required env variables
| Variable           | Example value                              |
| ------------------ | ------------------------------------------ |
| POSTGRESS_USERNAME | db-user                                    |
| POSTGRESS_PASSWORD | db-password                                |
| POSTGRESS_DB       | udagramdb                                  |
| POSTGRESS_HOST     | udagram.xxxxxx.us-east-1.rds.amazonaws.com |
| AWS_REGION         | us-east-1                                  |
| AWS_PROFILE        | default                                    |
| AWS_BUCKET         | udagrambucket                              |
| JWT_SECRET         | change-me                                  |
| DOCKER_USERNAME    | docker-user                                |
| URL                | http://localhost:8100 (frontend address)   |

Env variable `URL` is used in [server.ts](/udacity-c3-restapi-feed/src/server.ts) to enable CORS from
specified URL.
Env variable `AWS_PROFILE` is defining which AWS profile should be used from `~/.aws/credentials` file

Build docker images and run using docker-compose
```
docker-compose -f udacity-c3-deployment/docker/docker-compose-build.yaml build --parallel
docker-compose -f  udacity-c3-deployment/docker/docker-compose.yaml up -d
```

Stop services:
```
docker-compose -f  udacity-c3-deployment/docker/docker-compose.yaml down
```

### Automatic builds with Travis CI
[.travis.yml](/.travis.yml) file is using to describe

According to [Travis CI documentation](https://docs.travis-ci.com/user/docker/#pushing-a-docker-image-to-a-registry),
to push images into Docker registry, env variables `DOCKER_USERNAME` and `DOCKER_PASSWORD` should be specified.
They can be encrypted and included into `.travis.yml` using [CLI commands](https://docs.travis-ci.com/user/environment-variables#encrypting-environment-variables)
Access key can be generated in [Docker Hub account](https://hub.docker.com/settings/security)
```
travis encrypt DOCKER_USERNAME=<user> --add env.global
travis encrypt DOCKER_PASSWORD=<access-key> --add env.global
```

### Kubernetes deploy

- Create Kubernetes cluster using [Kubeone manual](https://github.com/kubermatic/kubeone/blob/master/docs/quickstart-aws.md)
- Apply kubernetes `*-secrets`, `*-deployment` , and `*-service` files
```
kubectl --kubeconfig udagram-kubeconfig apply -f env-configmap.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f env-secret.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f aws-secret.yaml

kubectl --kubeconfig udagram-kubeconfig apply -f backend-feed-deployment.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f backend-user-deployment.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f frontend-deployment.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f reverseproxy-deployment.yaml

kubectl --kubeconfig udagram-kubeconfig apply -f backend-feed-service.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f backend-user-service.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f frontend-service.yaml
kubectl --kubeconfig udagram-kubeconfig apply -f reverseproxy-service.yaml

```

- Check resources
```
kubectl --kubeconfig udagram-kubeconfig get pods
kubectl --kubeconfig udagram-kubeconfig get nodes
kubectl --kubeconfig udagram-kubeconfig get services
kubectl --kubeconfig udagram-kubeconfig get deployment
```

- Specify LoadBalancer EXTERNAL-IP as API URL in [Angular environments files](udacity-c3-frontend/src/environments)

- Destroy resources
```
kubectl --kubeconfig udagram-kubeconfig delete -f backend-feed-deployment.yaml
kubectl --kubeconfig udagram-kubeconfig delete -f backend-user-deployment.yaml
kubectl --kubeconfig udagram-kubeconfig delete -f frontend-deployment.yaml
kubectl --kubeconfig udagram-kubeconfig delete -f reverseproxy-deployment.yaml

kubectl --kubeconfig udagram-kubeconfig delete -f backend-feed-service.yaml
kubectl --kubeconfig udagram-kubeconfig delete -f backend-user-service.yaml
kubectl --kubeconfig udagram-kubeconfig delete -f frontend-service.yaml
kubectl --kubeconfig udagram-kubeconfig delete -f reverseproxy-service.yaml
```

- [Troubleshooting](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/):

```
kubectl --kubeconfig udagram-kubeconfig describe pods ${POD_NAME}
```

### Deployment from CI

- Specify required env variables
| Variable                       | Example value                                |
| ------------------------------ | -------------------------------------------- |
| KUBERNETES_SERVER              | https://xxx.elb.us-east-1.amazonaws.com:6443 |
| KUBERNETES_TOKEN               | ZXlKaGJHY2lPaUpT....VXpJMU5pSXNJbXRwWkNJN    |
| KUBERNETES_CLUSTER_CERTIFICATE | LS0tLS1CRUdJTiBD....RVJUSUZJQ0FURS0tLS0tC    |

- Create cluster access token for `travis` user and assign role
```
kubectl --kubeconfig udagram-kubeconfig create serviceaccount travis
kubectl --kubeconfig udagram-kubeconfig get serviceaccounts travis -o yaml

kubectl --kubeconfig udagram-kubeconfig create rolebinding travis \
  --clusterrole=admin \
  --serviceaccount=default:travis \
  --namespace=default

kubectl --kubeconfig udagram-kubeconfig get secrets
kubectl --kubeconfig udagram-kubeconfig describe secret travis-token-xxxx
kubectl --kubeconfig udagram-kubeconfig get secret travis-token-xxxx -o yaml
```
- Set Travis env variables
```
travis env set KUBERNETES_SERVER <kubernetes server url from udagram-kubeconfig>
travis env set KUBERNETES_TOKEN <token file content from prev command>
travis env set KUBERNETES_CLUSTER_CERTIFICATE <ca file content from prev command>
```
