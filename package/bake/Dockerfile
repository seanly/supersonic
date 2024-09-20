FROM seanly/toolset:nodejs-v20-1 AS webapp

COPY ./webapp /code
WORKDIR /code

RUN sh ./start-fe-prod.sh

FROM seanly/toolset:openjdk-8u372-b07-1 AS backend

COPY ./ /code
WORKDIR /code

RUN cp -r /maven-wrapper/mvnw /code/ \
    && cp -r /maven-wrapper/.mvn /code/

RUN bash ./mvnw clean package -DskipTests

FROM seanly/toolset:openjdk-8u372-b07-1 AS package


# Argument to pass in the supersonic version at build time
ARG SUPERSONIC_VERSION=0.9.8

COPY --from=backend /code/launchers/standalone/target/launchers-standalone-*.tar.gz /package/
COPY --from=webapp /code/supersonic-webapp.tar.gz /package/

WORKDIR /package/

RUN <<EOF

mkdir -p /package/dist/

BACKEND_PKG=$(ls -d /package/launchers-* |grep tar.gz)
tar --strip-components=1 -xzf $BACKEND_PKG -C /package/dist/

WEBAPP_PKG=$(ls -d /package/supersonic-webapp.tar.gz |grep tar.gz)
tar --strip-components=1 -xzf $WEBAPP_PKG -C /package/dist/

mv /package/dist/supersonic-webapp /package/dist/webapp

chmod +x /package/dist/bin/supersonic-daemon.sh
EOF

FROM seanly/toolset:openjdk-8u372-b07-1

# Set the working directory in the container
WORKDIR /usr/src/app

RUN apt-get update && \
    apt-get install -y default-mysql-client dnsutils 

COPY --from=package /package/dist /data/app

WORKDIR /data/app

# Expose the default port
EXPOSE 9080
CMD ["bash", "-c", "bin/supersonic-daemon.sh restart standalone prd && tail -f /dev/null"]