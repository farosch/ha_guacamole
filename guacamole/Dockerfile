ARG BUILD_FROM=ghcr.io/home-assistant/aarch64-base:latest

# Build guacamole-server (guacd + protocol plugins) from source
FROM $BUILD_FROM AS build_guacd

SHELL ["/bin/ash", "-o", "pipefail", "-c"]

RUN ALP_VER=$(cut -d. -f1,2 /etc/alpine-release) && \
  echo "https://dl-cdn.alpinelinux.org/alpine/v${ALP_VER}/main" > /etc/apk/repositories && \
  echo "https://dl-cdn.alpinelinux.org/alpine/v${ALP_VER}/community" >> /etc/apk/repositories && \
  apk add --no-cache \
  build-base \
  autoconf automake libtool \
  pkgconfig cmake \
  git curl tar \
  zlib-dev openssl-dev \
  freerdp-dev \
  libvncserver-dev \
  libssh2-dev \
  cairo-dev \
  pango-dev \
  harfbuzz-dev \
  libpng-dev \
  libjpeg-turbo-dev \
  libwebp-dev \
  fontconfig ttf-dejavu

# Build and install libtelnet (required for telnet protocol)
ENV LIBTELNET_VERSION=0.23
RUN set -e && \
  curl -fsSL -o /tmp/libtelnet.tar.gz "https://github.com/kinetiknz/libtelnet/archive/refs/tags/${LIBTELNET_VERSION}.tar.gz" && \
  mkdir -p /tmp/libtelnet && tar -xzf /tmp/libtelnet.tar.gz -C /tmp/libtelnet --strip-components=1 && \
  cd /tmp/libtelnet && \
  ./configure --prefix=/usr && \
  make -j$(nproc) && make install && \
  rm -rf /tmp/libtelnet /tmp/libtelnet.tar.gz

# Build and install guacamole-server (guacd)
ENV GUAC_VERSION=1.6.0
RUN set -e && \
  curl -fsSL -o /tmp/guac-server.tar.gz "https://archive.apache.org/dist/guacamole/${GUAC_VERSION}/source/guacamole-server-${GUAC_VERSION}.tar.gz" && \
  mkdir -p /tmp/guac-server && tar -xzf /tmp/guac-server.tar.gz -C /tmp/guac-server --strip-components=1 && \
  cd /tmp/guac-server && \
  ./configure --prefix=/usr && \
  make -j$(nproc) && make install && \
  ldconfig || true && \
  rm -rf /tmp/guac-server /tmp/guac-server.tar.gz

# Final runtime image
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
  libwebp \
  fontconfig \
  ttf-dejavu \
  busybox-extras \
  openssl \
  mariadb mariadb-client \
  jq \
  curl \
  tar \
  ca-certificates

# Copy guacd and protocol plugins from build stage
COPY --from=build_guacd /usr/sbin/guacd /usr/sbin/guacd
COPY --from=build_guacd /usr/lib/guacamole /usr/lib/guacamole
COPY --from=build_guacd /usr/lib/libguac* /usr/lib/
COPY --from=build_guacd /usr/lib/libtelnet* /usr/lib/

# Verify protocol client libraries are present
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
