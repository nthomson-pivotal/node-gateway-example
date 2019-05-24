# Node Gateway Example

This repository illustrate an example of using Spring Cloud Gateway to proxy requests to Node applications running in CloudFoundry.

## Usage

First, build the Spring Cloud Gateway application:

```
cd spring-cloud-gateway
./mvnw package
```

Then, from the root directory of the repository `push` the applications to CloudFoundry:

```
cf push
```

This will push 3 applications:
- Sample Node service A
- Sample Node service B
- Spring Cloud Gateway proxy

Once the push has completed, get the random route for the `node-sample-gateway` application:

```
$ cf app node-sample-gateway

Showing health and status for app node-sample-gateway in org test / space test as admin...

name:              node-sample-gateway
requested state:   started
routes:            node-sample-gateway-excellent-hippopotamus.dev.cfdev.sh
last uploaded:     Thu 23 May 20:28:57 PDT 2019
stack:             cflinuxfs3
buildpacks:        client-certificate-mapper=1.8.0_RELEASE container-security-provider=1.16.0_RELEASE
                   java-buildpack=v4.16.1-https://github.com/cloudfoundry/java-buildpack.git#41b8ff8 java-main java-opts java-security jvmkill-agent=1.16.0_RELEASE
                   open-jdk-like-j...
```

Allow container-to-container communication with the following network policies:

```
$ cf add-network-policy node-sample-gateway --destination-app node-sample-a --protocol tcp --port 8080
Adding network policy to app node-sample-gateway in org test / space test as admin...
OK

$ cf add-network-policy node-sample-gateway --destination-app node-sample-b --protocol tcp --port 8080
Adding network policy to app node-sample-gateway in org test / space test as admin...
OK
```

Then you can use something like `curl` or a browser to access `/service-a` or `/service-b`:

```
$ curl http://node-sample-gateway-excellent-hippopotamus.dev.cfdev.sh/service-b
<!DOCTYPE html><html><head><title>Service B</title><link rel="stylesheet" href="/stylesheets/style.css"></head><body><h1>Service B</h1><p>This is Service B</p></body></html>
```

## Why?

Similar functionality as illustrated here could also be accomplished using Nginx or Haproxy instead of Spring Cloud Gateway. So why would you want to use the latter?

- Its a Spring Boot application, so it feeds in to CloudFoundry sub-systems like logging and monitoring
- You can transform requests and/or responses (add/remove headers etc)
- Its possible to cache responses in an external cache like Pivotal Cloud Cache or Redis
- Circuit breakers can be built in for fallback mechanics