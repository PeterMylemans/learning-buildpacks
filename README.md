# Learning Build Packs

In this example, I'll walk you through using a build pack (https://buildpacks.io/) for a Java application.

## Requirements:

Make sure that you have a recent version of **JDK** installed >= Java 11.

Make sure that you have **docker** installed: e.g. Docker Desktop on Mac/Windows.

You can use the builtin maven in your IDE or use the maven wrapper provided in this project.

## Build using Spring Boot

1. Build the application jar
2. Use Spring Boot's builtin support for generating OCI compliant docker images using build packs.
   
You can find a good starting guide on https://spring.io/guides/gs/spring-boot-docker/.

Note that the default uses paketo (https://paketo.io/) base builder image.

This has great support for Java images.

```shell
./mvnw clean spring-boot:build-image
```

## Run the container

Let's test it out!

```shell
docker run -it --rm docker.io/library/demo:0.0.1-SNAPSHOT
```

Looks great!
By default the image has helpers to:
- precalculate the right memory settings for java applications
- generate a truststore for the JRE that contains all the system certificates from your chosen stack (Ubuntu Bionic in 2021)
- run a JVM killer in case out of memory errors result in a broken runtime.


## Run with local timezone

But something seems off: the image is running in UTC, while we are in < insert your timezone here >.

By default the runtime image comes with timezone data, so fixing this is as easy as setting the `TZ` environment variable to your timezone.

Mine is Europe/Brussels, so the improved command becomes:

```shell
docker run -it --rm -e TZ=Europe/Brussels --rm docker.io/library/demo:0.0.1-SNAPSHOT
```

## Run with service bindings

In many cases you'll want to inject secrets or configurations at runtime that are peculiar to your deployment environment.

For example:
- certificates in case your organization runs its own private CA for internal services
- urls and credentials to databases
- ...

You can solve this by using service bindings (https://github.com/k8s-service-bindings/spec).

A typical projected application structure is:

```plain
$SERVICE_BINDING_ROOT
├── a-name-of-your-chosing
│   ├── type <-- must contain a specific type: e.g. database
│   ├── provider <-- in this example, the type database requires a provider, uri, username and password
│   ├── uri
│   ├── username
│   └── password
```

We'll be using a service binding of type `ca-certificates` that maps any certificate in the same directory.

The `SERVICE_BINDING_ROOT` environment variable can be chosen as you want, but `/bindings` seems like a good default.
You just have to make sure to mount your secrets in the same directory tree that you chose.

```shell
docker run -it --rm \
    -v $(pwd)/bindings:/bindings \
    -e SERVICE_BINDING_ROOT=/bindings \
    -e TZ=Europe/Brussels \
    docker.io/library/demo:0.0.1-SNAPSHOT
```
