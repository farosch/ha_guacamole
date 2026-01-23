ARG BUILD_FROM=ghcr.io/home-assistant/aarch64-base:latest
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
  fontconfig \
  ttf-dejavu \
  busybox-extras \
  guacamole-server \
  mariadb mariadb-client \
  jq \
  curl \
  tar \
  ca-certificates

# Ensure protocol client libraries are present in the default directory
RUN set -e && mkdir -p /usr/lib/guacamole && \
  echo "[build] Contents of /usr/lib/guacamole:" && ls -la /usr/lib/guacamole || true

# Verify protocol plugins and FreeRDP plugin directory (diagnostics only)
RUN set -e && echo "[build] Checking FreeRDP plugin directory" && \
  if [ -d /usr/lib/freerdp3 ]; then ls -la /usr/lib/freerdp3; elif [ -d /usr/lib/freerdp2 ]; then ls -la /usr/lib/freerdp2; else echo "[build] WARNING: FreeRDP plugins directory not found"; fi

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
