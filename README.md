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
