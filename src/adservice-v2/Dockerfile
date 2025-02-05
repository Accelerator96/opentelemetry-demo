FROM eclipse-temurin:17-jdk AS builder

WORKDIR /app/

COPY ./src/adservice-v2/  ./
RUN ./mvnw --batch-mode clean package -DskipTests

# -----------------------------------------------------------------------------

FROM docker.m.daocloud.io/library/openjdk:11.0.15-jre

WORKDIR /app/
ARG version=1.23.0

COPY --from=builder /app/target/adservice-v2-1.0-SNAPSHOT.jar ./
COPY --from=builder /app/jmx_prometheus_javaagent-0.17.0.jar ./
COPY --from=builder /app/prometheus-jmx-config.yaml ./

ADD https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$version/opentelemetry-javaagent.jar /usr/src/app/opentelemetry-javaagent.jar

RUN chmod 644 /usr/src/app/opentelemetry-javaagent.jar

ENV OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317 \
    OTEL_RESOURCE_ATTRIBUTES=service.name=adservice-springcloud \
    JAVA_TOOL_OPTIONS=-javaagent:/usr/src/app/opentelemetry-javaagent.jar \
    JAVA_OPTS='-Dspring.cloud.nacos.config.enabled=false -Dspring.randomError=false' \
    AD_SERVICE_PORT=8080

EXPOSE 8081 8999 8080 8888

ENTRYPOINT java $JAVA_OPTS -jar adservice-v2-1.0-SNAPSHOT.jar
