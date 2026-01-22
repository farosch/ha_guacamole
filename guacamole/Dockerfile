ARG BUILD_FROM=ghcr.io/home-assistant/aarch64-addon-base:latest
FROM $BUILD_FROM

ENV GUAC_VERSION=1.5.5 \
    GUAC_HOME=/data/guacamole \
    TOMCAT_WEBAPPS=/usr/share/tomcat/webapps \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2

SHELL ["/bin/ash", "-o", "pipefail", "-c"]

RUN apk add --no-cache \
  openjdk17-jre-headless \
  tomcat-native \
  tomcat \
  guacamole-server \
  mariadb mariadb-client \
  jq \
  curl \
  tar \
  ca-certificates

# Download Guacamole WAR and JDBC MySQL auth extension
RUN mkdir -p /opt/guacamole && \
    curl -fsSL "https://downloads.apache.org/guacamole/${GUAC_VERSION}/binary/guacamole-${GUAC_VERSION}.war" -o "${TOMCAT_WEBAPPS}/guacamole.war" && \
    curl -fsSL "https://downloads.apache.org/guacamole/${GUAC_VERSION}/binary/guacamole-auth-jdbc-${GUAC_VERSION}.tar.gz" -o \
      "/opt/guacamole/guacamole-auth-jdbc-${GUAC_VERSION}.tar.gz" && \
    tar -xzf "/opt/guacamole/guacamole-auth-jdbc-${GUAC_VERSION}.tar.gz" -C /opt/guacamole && \
    # MySQL JDBC driver
    curl -fsSL "https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.3.0/mysql-connector-j-8.3.0.jar" -o \
      "/opt/guacamole/mysql-connector-j.jar"

# Create directories used by services and persistence
RUN mkdir -p ${GUAC_HOME}/extensions ${GUAC_HOME}/lib /data/mysql /run/mysqld /etc/services.d /etc/cont-init.d

# Copy rootfs (init and service scripts)
COPY rootfs /

EXPOSE 8080

# s6-overlay (from base) will start services
