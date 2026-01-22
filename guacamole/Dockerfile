ARG BUILD_FROM=ghcr.io/home-assistant/aarch64-base:latest
FROM guacamole/guacd:1.6.0 AS guacd_src

# Extract guacd binary and protocol/client libraries from the official image
RUN set -e && mkdir -p /tmp/export/guacamole && \
  # Copy guacd binary from common locations
  for b in /usr/local/sbin/guacd /usr/sbin/guacd /usr/bin/guacd; do \
    if [ -x "$b" ]; then cp "$b" /tmp/export/guacd; break; fi; \
  done && \
  # Copy any existing guacamole plugin directories
  for d in /usr/local/lib/guacamole /usr/lib/guacamole /usr/lib/aarch64-linux-gnu/guacamole /opt/guacamole/lib/guacamole; do \
    if [ -d "$d" ]; then cp -a "$d"/* /tmp/export/guacamole/; fi; \
  done && \
  # Find and copy protocol client libraries wherever they are
  find /usr /opt -type f -name 'libguac-client-*.so' -exec cp -t /tmp/export/guacamole {} + && \
  # Copy core libguac libraries
  for l in /usr/local/lib/libguac* /usr/lib/libguac* /usr/lib/aarch64-linux-gnu/libguac*; do \
    if ls $l >/dev/null 2>&1; then cp $l /tmp/export/; fi; \
  done

FROM $BUILD_FROM

ENV GUAC_VERSION=1.6.0 \
  GUAC_HOME=/data/guacamole \
  GUACAMOLE_HOME=/data/guacamole \
  TOMCAT_VERSION=9.0.85 \
  TOMCAT_DIR=/opt/tomcat \
  TOMCAT_WEBAPPS=/opt/tomcat/webapps \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2

SHELL ["/bin/ash", "-o", "pipefail", "-c"]

RUN ALP_VER=$(cut -d. -f1,2 /etc/alpine-release) && \
  echo "https://dl-cdn.alpinelinux.org/alpine/v${ALP_VER}/main" > /etc/apk/repositories && \
  echo "https://dl-cdn.alpinelinux.org/alpine/v${ALP_VER}/community" >> /etc/apk/repositories && \
  apk add --no-cache \
  openjdk17-jre-headless \
  libssh2 \
  freerdp \
  libvncserver \
  cairo \
  pango \
  glib \
  harfbuzz \
  libpng \
  libjpeg-turbo \
  guacamole-server \
  mariadb mariadb-client \
  jq \
  curl \
  tar \
  ca-certificates

# Copy guacd and protocol plugins from official image (normalized into /tmp/export)
COPY --from=guacd_src /tmp/export /opt/guacd-export
RUN set -e && \
  if [ -f /opt/guacd-export/guacd ]; then install -D /opt/guacd-export/guacd /usr/local/sbin/guacd; fi && \
  if [ -d /opt/guacd-export/guacamole ]; then mkdir -p /usr/lib/guacamole && cp -a /opt/guacd-export/guacamole/. /usr/lib/guacamole/; fi && \
  if ls /opt/guacd-export/libguac* >/dev/null 2>&1; then cp /opt/guacd-export/libguac* /usr/lib/; fi && \
  rm -rf /opt/guacd-export

# Download Tomcat
RUN rm -rf ${TOMCAT_DIR} && \
    curl -fsSL "https://archive.apache.org/dist/tomcat/tomcat-9/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz" -o /tmp/tomcat.tar.gz && \
    tar -xzf /tmp/tomcat.tar.gz -C /opt && \
    mv /opt/apache-tomcat-${TOMCAT_VERSION} ${TOMCAT_DIR} && \
    rm -f /tmp/tomcat.tar.gz

# Download Guacamole WAR and JDBC MySQL auth extension (use archive.apache.org; write to /tmp then move)
RUN mkdir -p /opt/guacamole ${TOMCAT_WEBAPPS} && \
    apk add --no-cache wget && \
    wget -q "https://archive.apache.org/dist/guacamole/${GUAC_VERSION}/binary/guacamole-${GUAC_VERSION}.war" -O /tmp/guacamole.war && \
    test -s /tmp/guacamole.war && mv /tmp/guacamole.war "${TOMCAT_WEBAPPS}/guacamole.war" && \
    wget -q "https://archive.apache.org/dist/guacamole/${GUAC_VERSION}/binary/guacamole-auth-jdbc-${GUAC_VERSION}.tar.gz" -O \
      "/opt/guacamole/guacamole-auth-jdbc-${GUAC_VERSION}.tar.gz" && \
    test -s "/opt/guacamole/guacamole-auth-jdbc-${GUAC_VERSION}.tar.gz" && \
    tar -xzf "/opt/guacamole/guacamole-auth-jdbc-${GUAC_VERSION}.tar.gz" -C /opt/guacamole && \
    # MySQL JDBC driver
    wget -q "https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.3.0/mysql-connector-j-8.3.0.jar" -O \
      "/opt/guacamole/mysql-connector-j.jar" && \
    test -s "/opt/guacamole/mysql-connector-j.jar"

# Create directories used by services and persistence
RUN mkdir -p ${GUAC_HOME}/extensions ${GUAC_HOME}/lib /data/mysql /run/mysqld /etc/services.d /etc/cont-init.d

# Copy rootfs (init and service scripts)
COPY rootfs /

# Ensure init and service scripts are executable
RUN find /etc/cont-init.d -type f -exec chmod 0755 {} \; && \
  find /etc/services.d -type f -name run -exec chmod 0755 {} \; && \
  sed -i 's/\r$//' /etc/cont-init.d/* /etc/services.d/*/run

EXPOSE 8080

# s6-overlay (from base) will start services
