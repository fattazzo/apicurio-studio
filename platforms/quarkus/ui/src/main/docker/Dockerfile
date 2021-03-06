####
# This Dockerfile is used in order to build a container that runs the Quarkus application in JVM mode
#
# Before building the docker image run:
#
# mvn package
#
# Then, build the image with:
#
# docker build -f src/main/docker/Dockerfile.jvm -t quarkus/apicurio-studio-platforms-quarkus-ui-jvm .
#
# Then run the container using:
#
# docker run -i --rm -p 8080:8080 quarkus/apicurio-studio-platforms-quarkus-ui-jvm
#
###
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.1

ARG JAVA_PACKAGE=java-11-openjdk-headless
ARG RUN_JAVA_VERSION=1.3.5

ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en'
# Configure the JAVA_OPTIONS, you can add -XshowSettings:vm to also display the heap size.
ENV JAVA_TOOL_OPTIONS=-Djava.net.preferIPv4Stack=true
ENV APICURIO_KC_AUTH_URL=http://localhost:8180/auth
ENV APICURIO_KC_REALM=apicurio
ENV APICURIO_KC_CLIENT_ID=apicurio
ENV APICURIO_KC_SSL_REQUIRED=NONE
ENV APICURIO_KC_DISABLE_TRUST_MANAGER=true
ENV APICURIO_PORT_OFFSET=0
ENV APICURIO_LOGGING_LEVEL=INFO
ENV APICURIO_UI_HUB_API_URL=https://localhost:8443/api
ENV APICURIO_UI_EDITING_URL=https://localhost:8443/ws
ENV APICURIO_MIN_HEAP=512m
ENV APICURIO_MAX_HEAP=2048m
ENV QUARKUS_OIDC_AUTH_SERVER=http://localhost:8180/auth/realms/apicurio
ENV QUARKUS_OIDC_CLIENT_ID=apicurio-studio
ENV QUARKUS_OIDC_APPLICATION_TYPE=web-app

# Install java and the run-java script
# Also set up permissions for user `1001`
RUN microdnf install curl ca-certificates ${JAVA_PACKAGE} \
    && microdnf update \
    && microdnf clean all \
    && mkdir /deployments \
    && chown 1001 /deployments \
    && chmod "g+rwX" /deployments \
    && chown 1001:root /deployments \
    && curl https://repo1.maven.org/maven2/io/fabric8/run-java-sh/${RUN_JAVA_VERSION}/run-java-sh-${RUN_JAVA_VERSION}-sh.sh -o /deployments/run-java.sh \
    && chown 1001 /deployments/run-java.sh \
    && chmod 540 /deployments/run-java.sh \
    && echo "securerandom.source=file:/dev/urandom" >> /etc/alternatives/jre/lib/security/java.security

ENV JAVA_OPTIONS="-Dquarkus.http.host=0.0.0.0 \
 -Djava.util.logging.manager=org.jboss.logmanager.LogManager \
 -Xms${APICURIO_MIN_HEAP} \
   -Xmx${APICURIO_MAX_HEAP} \
   -Dquarkus.port.offset=${APICURIO_PORT_OFFSET} \
   -Dquarkus.oidc.auth-server-url=${QUARKUS_OIDC_AUTH_SERVER} \
   -Dquarkus.oidc.client-id=${QUARKUS_OIDC_CLIENT_ID} \
   -Dquarkus.oidc.application-type=${QUARKUS_OIDC_APPLICATION_TYPE} \
   -Dapicurio.kc.auth.rootUrl=${APICURIO_KC_AUTH_URL} \
   -Dapicurio.kc.auth.realm=${APICURIO_KC_REALM} \
   -Dapicurio-ui.hub-api.url=${APICURIO_UI_HUB_API_URL} \
   -Dapicurio-ui.editing.url=${APICURIO_UI_EDITING_URL} \
   -Dquarkus.log.level=${APICURIO_LOGGING_LEVEL}"

COPY target/lib/* /deployments/lib/
COPY target/*-runner.jar /deployments/app.jar

EXPOSE 8080
USER 1001

ENTRYPOINT [ "/deployments/run-java.sh" ]