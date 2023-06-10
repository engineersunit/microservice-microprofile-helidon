# helidon-mp

Minimal Helidon MP project suitable to start from scratch.

## Build and run


With JDK17+
```bash
mvn package
java -jar target/helidon-mp.jar
```

## Exercise the application
```
curl -X GET http://localhost:8080/simple-greet
{"message":"Hello World!"}
```

```
curl -X GET http://localhost:8080/greet
{"message":"Hello World!"}

curl -X GET http://localhost:8080/greet/Joe
{"message":"Hello Joe!"}

curl -X PUT -H "Content-Type: application/json" -d '{"greeting" : "Hola"}' http://localhost:8080/greet/greeting

curl -X GET http://localhost:8080/greet/Jose
{"message":"Hola Jose!"}
```



## Try metrics

```
# Prometheus Format
curl -s -X GET http://localhost:8080/metrics
# TYPE base:gc_g1_young_generation_count gauge
. . .

# JSON Format
curl -H 'Accept: application/json' -X GET http://localhost:8080/metrics
{"base":...
. . .
```



## Try health

```
curl -s -X GET http://localhost:8080/health
{"outcome":"UP",...

```



## Building a Native Image

Make sure you have GraalVM locally installed:

```
$GRAALVM_HOME/bin/native-image --version
```

Build the native image using the native image profile:

```
mvn package -Pnative-image
```

This uses the helidon-maven-plugin to perform the native compilation using your installed copy of GraalVM. It might take a while to complete.
Once it completes start the application using the native executable (no JVM!):

```
./target/helidon-mp
```

Yep, it starts fast. You can exercise the applicationâ€™s endpoints as before.


## Building the Docker Image

```
docker build -t helidon-mp .
```

## Running the Docker Image

```
docker run --rm -p 8080:8080 helidon-mp:latest
```

Exercise the application as described above.
                                

## Building a Custom Runtime Image

Build the custom runtime image using the jlink image profile:

```
mvn package -Pjlink-image
```

This uses the helidon-maven-plugin to perform the custom image generation.
After the build completes it will report some statistics about the build including the reduction in image size.

The target/helidon-mp-jri directory is a self contained custom image of your application. It contains your application,
its runtime dependencies and the JDK modules it depends on. You can start your application using the provide start script:

```
./target/helidon-mp-jri/bin/start
```

Class Data Sharing (CDS) Archive
Also included in the custom image is a Class Data Sharing (CDS) archive that improves your application startup
performance and in-memory footprint. You can learn more about Class Data Sharing in the JDK documentation.

The CDS archive increases your image size to get these performance optimizations. It can be of significant size (tens of MB).
The size of the CDS archive is reported at the end of the build output.

If you rather have a smaller image size (with a slightly increased startup time) you can skip the creation of the CDS
archive by executing your build like this:

```
mvn package -Pjlink-image -Djlink.image.addClassDataSharingArchive=false
```

For more information on available configuration options see the helidon-maven-plugin documentation.

## Enabling Tracing With OpenTelemetry & Jaeger

### References

[Helidon CLI](https://helidon.io/docs/v3/#/about/cli)

[Maven repo: helidon-tracing-jaeger](https://mvnrepository.com/artifact/io.helidon.tracing/helidon-tracing-jaeger)

[OpenTracing](https://opentracing.io/)

[OpenTelemetry](https://opentelemetry.io/)

[OpenTelemetry  Migrating from OpenTracing](https://opentelemetry.io/docs/migration/opentracing/)

[Helidon v3 Tracing setup](https://helidon.io/docs/v3/#/mp/tracing#jaeger-tracing)

[CNCF Archives the OpenTracing Project](https://www.cncf.io/blog/2022/01/31/cncf-archives-the-opentracing-project/)

### pom.xml
```xml
<dependency>
    <groupId>io.helidon.microprofile.tracing</groupId>
    <artifactId>helidon-microprofile-tracing</artifactId>
    <version>3.2.0</version>
</dependency>

<dependency>
    <groupId>io.helidon.tracing</groupId>
    <artifactId>helidon-tracing-jaeger</artifactId>
</dependency>
```
### microprofile-config.properties
>Jaeger changed its client implementation, 
> so some Jaeger settings exposed by earlier releases of Helidon are no longer available. Please note the currently-supported settings in the table below.
---
>The the Jaeger OpenTelemetry client uses port 14250, but you can override this value if needed. The default is defined by each tracing integration.

[Jaeger Client Sampling Configuration](https://www.jaegertracing.io/docs/1.46/sampling/#client-sampling-configuration)

```properties
tracing.service=helidon-greet
tracing.sampler-type=const
tracing.sampler-param=1
tracing.enabled=true
tracing.protocol=http
tracing.host=localhost
tracing.port=14250
```



                                
