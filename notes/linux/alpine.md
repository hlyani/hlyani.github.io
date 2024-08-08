# alpine

[https://hub.docker.com/_/alpine](https://hub.docker.com/_/alpine)

```
docker pull alpine:3.20.2
docker pull python:alpine3.20
docker pull golang:alpine3.20
```

```
FROM alpine:3.14
RUN apk add --no-cache mysql-client
ENTRYPOINT ["mysql"]
```