FROM ubuntu:jammy as builder
LABEL maintainer="Chinthaka <axcelor@libresoft.io>"

RUN set -ex \
	&& DEBIAN_FRONTEND=noninteractive apt-get update \
	&& DEBIAN_FRONTEND=noninteractive apt-get upgrade -y \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -y curl dpkg-dev openjdk-11-jdk gcc libapr1-dev libssl-dev make \
	&& rm -rf /var/lib/apt/lists/*

# jdk
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64
ENV PATH "${JAVA_HOME}/bin:${PATH}"

# tomcat
ENV CATALINA_HOME /usr/local/tomcat
ENV CATALINA_BASE /var/lib/tomcat

RUN set -ex \
	&& export TOMCAT_MAJOR="9" \
	&& export TOMCAT_VERSION="9.0.90" \
	&& export TOMCAT_ESUM="e77b47d7ded86da81018d38c4f728f5f804c1a65bb941a138a7989b69c859031e88d113ccf4fc3a409062ee24511fa5ccf15dfad333f570838ee2a36dae23e19" \
	&& export TOMCAT_PKG="apache-tomcat-$TOMCAT_VERSION.tar.gz" \
	&& mkdir -p $CATALINA_HOME \
	&& mkdir -p $CATALINA_BASE \
	&& cd $CATALINA_HOME \
	&& curl -L -O "https://dlcdn.apache.org/tomcat/tomcat-9/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz" \
	&& echo "$TOMCAT_ESUM $TOMCAT_PKG" | sha512sum -c - \
	&& tar -xvf $TOMCAT_PKG --strip-components=1 \
	&& rm -f $TOMCAT_PKG \
	&& rm -f $TOMCAT_HOME/bin/*.bat \
	&& chmod 755 $CATALINA_HOME/bin \
	&& chmod 755 $CATALINA_HOME/lib \
	&& chmod 755 $CATALINA_HOME/conf \
	&& chmod 644 $CATALINA_HOME/bin/* \
	&& chmod 644 $CATALINA_HOME/lib/* \
	&& chmod 644 $CATALINA_HOME/conf/* \
	&& chmod 755 $CATALINA_HOME/bin/*.sh \
	&& chown root:$TOMCAT_GROUP $CATALINA_HOME/conf/* \
	&& mkdir -p $CATALINA_BASE/conf \
	&& mkdir -p $CATALINA_BASE/temp \
	&& mkdir -p $CATALINA_BASE/webapps \
	&& cp $CATALINA_HOME/conf/tomcat-users.xml $CATALINA_BASE/conf/ \
	&& cp $CATALINA_HOME/conf/logging.properties $CATALINA_BASE/conf/ \
	&& cp $CATALINA_HOME/conf/server.xml $CATALINA_BASE/conf/ \
	&& cp $CATALINA_HOME/conf/web.xml $CATALINA_BASE/conf/ \
	&& sed -i 's/directory="logs"/directory="\/var\/log\/tomcat"/g' $CATALINA_BASE/conf/server.xml \
	&& sed -i 's/\${catalina\.base}\/logs/\/var\/log\/tomcat/g' $CATALINA_BASE/conf/logging.properties

# tomcat-native lib path
ENV TOMCAT_NATIVE_LIBDIR $CATALINA_HOME/native-jni-lib
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH:+$LD_LIBRARY_PATH:}$TOMCAT_NATIVE_LIBDIR

# build tomcat-native
RUN set -ex \
	&& export NATIVE_BUILD_DIR="$(mktemp -d)" \
	&& tar -xvf $CATALINA_HOME/bin/tomcat-native.tar.gz -C "$NATIVE_BUILD_DIR" --strip-components=1 \
	&& cd $NATIVE_BUILD_DIR/native \
	&& ./configure \
		--build="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
		--libdir="$TOMCAT_NATIVE_LIBDIR" \
		--prefix="$CATALINA_HOME" \
		--with-apr="$(which apr-1-config)" \
		--with-java-home="$JAVA_HOME" \
		--with-ssl=yes \
	&& make -j "$(nproc)" \
	&& make install \
	&& rm -f $CATALINA_HOME/bin/*.bat \
	&& rm -f $CATALINA_HOME/bin/*.tar.gz \
	&& rm -rf $NATIVE_BUILD_DIR

## end of builder stage

FROM  ubuntu:jammy
LABEL maintainer="Chinthaka Deshapriyiya  <axcelor@ceee.dev>"

RUN set -ex \
	&& DEBIAN_FRONTEND=noninteractive apt-get update \
	&& DEBIAN_FRONTEND=noninteractive apt-get upgrade -y \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -y openjdk-11-jdk supervisor gosu postgresql-14 libapr1 nginx libnginx-mod-http-xslt-filter \
        && DEBIAN_FRONTEND=noninteractive apt-get install -y libnginx-mod-http-geoip2 libnginx-mod-http-image-filter  gettext-base git-core nodejs yarn  \
        && rm -rf /var/lib/apt/lists/*

# jdk
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64
ENV PATH "${JAVA_HOME}/bin:${PATH}"

# update locale
RUN set -ex && \
	echo 'en_GB.UTF-8 UTF-8' >> /etc/locale.gen && \
	echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen
ENV LANG en_US.utf8

# tomcat
ENV TOMCAT_USER tomcat
ENV TOMCAT_GROUP tomcat

ENV CATALINA_HOME /usr/local/tomcat
ENV CATALINA_BASE /var/lib/tomcat

# copy from builder
COPY --from=builder $CATALINA_HOME $CATALINA_HOME
COPY --from=builder $CATALINA_BASE $CATALINA_BASE

RUN set -ex \
	&& addgroup --system "$TOMCAT_GROUP" --quiet \
	&& adduser \
		--system --home "$CATALINA_BASE" --no-create-home \
		--ingroup "$TOMCAT_GROUP" --disabled-password --shell /bin/false "$TOMCAT_USER"

RUN set -ex \
	&& update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 20000 \
	&& update-alternatives --install /usr/bin/javac javac $JAVA_HOME/bin/javac 20000 \
	&& update-alternatives --install /usr/bin/jar jar $JAVA_HOME/bin/jar 20000 \
	&& update-alternatives --install /usr/bin/tomcat tomcat $CATALINA_HOME/bin/catalina.sh 20000 \
	&& update-alternatives --install /usr/bin/tomcat-digest tomcat-digest $CATALINA_HOME/bin/digest.sh 20000 \
	&& update-alternatives --install /usr/bin/tomcat-tool-wrapper tomcat-tool-wrapper $CATALINA_HOME/bin/tool-wrapper.sh 20000 \
	&& chown root:$TOMCAT_GROUP $CATALINA_HOME/conf/* \
	&& mkdir -p /var/log/tomcat \
	&& chown -R $TOMCAT_USER:$TOMCAT_GROUP $CATALINA_BASE \
	&& chown -R $TOMCAT_USER:$TOMCAT_GROUP /var/log/tomcat

# tomcat-native lib path
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH:+$LD_LIBRARY_PATH:}$CATALINA_HOME/native-jni-lib

# postgres
ENV POSTGRES_USER axelor
ENV POSTGRES_PASSWORD axelor
ENV POSTGRES_DB axelor

ENV PATH $PATH:/usr/lib/postgresql/14/bin
ENV PGDATA /var/lib/postgresql/14/main

RUN set -ex \
	&& echo "host all all all md5" >> /etc/postgresql/14/main/pg_hba.conf \
	&& echo "listen_addresses='localhost'" >> /etc/postgresql/14/main/postgresql.conf \
	&& rm -rf /var/lib/postgresql/14/main \
	&& mkdir -p /var/lib/postgresql/14 \
	&& chown -R postgres:postgres /var/lib/postgresql

# nginx
ENV NGINX_HOST localhost
ENV NGINX_PORT 443

RUN mkdir -p /etc/nginx/conf.d
RUN mkdir -p /etc/nginx/certs
RUN mkdir -p /etc/nginx/conf.d.templates

COPY nginx.conf /etc/nginx/conf.d.templates/
COPY nginx-ssl.conf /etc/nginx/conf.d.templates/

VOLUME ["/var/lib/tomcat", "/var/lib/postgresql", "/var/log/tomcat", "/var/log/postgresql"]

EXPOSE  80 443 8080 5432

COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh && mkdir /docker-entrypoint-initdb.d

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["start"]
