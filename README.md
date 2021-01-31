# Learning Build Packs

## Build using Spring Boot

```shell
mvn clean spring-boot:build-image
```

## Run the container

```shell
docker run -it --rm docker.io/library/demo:0.0.1-SNAPSHOT
```

## Run with local timezone

```shell
docker run -it --rm -e TZ=Europe/Brussels --rm docker.io/library/demo:0.0.1-SNAPSHOT
```

## Run with service bindings

```shell
docker run -it --rm \
    -v $(pwd)/bindings:/bindings \
    -e SERVICE_BINDING_ROOT=/bindings \
    -e TZ=Europe/Brussels \
    docker.io/library/demo:0.0.1-SNAPSHOT
```
