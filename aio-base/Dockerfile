FROM eclipse-temurin:11-focal as builder

RUN  apt-get update -y &&  apt-get upgrade -y \
     &&  apt-get install -y curl dpkg-dev gcc libapr1-dev libssl-dev make \
     && rm -rf /var/lib/apt/lists/*

# jdk
ENV JAVA_HOME /opt/java/openjdk

# tomcat
ENV CATALINA_HOME /usr/local/tomcat
ENV CATALINA_BASE /var/lib/tomcat

RUN set -ex \
	&& export TOMCAT_MAJOR="9" \
	&& export TOMCAT_VERSION="9.0.85" \
	&& export TOMCAT_ESUM="06e239d15ff7b72017c1d0752ddb1be4651374f7c1391631ec5619f4981cb2911267bc6b044d6c71a2a74738f70d433b96418951439848121f1d874862ddd3de" \
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

FROM eclipse-temurin:11-focal
LABEL maintainer="Chinthaka  <axcelor@libresoft.io>"

RUN  apt-get update -y && apt-get upgrade -y && \
     apt-get install -y gnupg dirmngr apt-transport-https locales \
     apt-get install -y supervisor gosu postgresql-12 libapr1 nginx libnginx-mod-http-xslt-filter nginx-module-geoip nginx-module-image-filter nginx-module-njs gettext-base git-core nodejs yarn \     rm -rf /var/lib/apt/lists/*

# update locale
RUN set -ex && \
	echo 'en_GB.UTF-8 UTF-8' >> /etc/locale.gen && \
	echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen
ENV LANG en_US.utf8

# jdk
ENV JAVA_HOME /opt/java/openjdk

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

ENV PATH $PATH:/usr/lib/postgresql/12/bin
ENV PGDATA /var/lib/postgresql/12/main

RUN set -ex \
	&& echo "host all all all md5" >> /etc/postgresql/12/pg_hba.conf \
	&& echo "listen_addresses='localhost'" >> /etc/postgresql/12/main/postgresql.conf \
	&& rm -rf /var/lib/postgresql/12/main \
	&& mkdir -p /var/lib/postgresql/12 \
	&& chown -R postgres:postgres /var/lib/postgresql

# nginx
ENV NGINX_HOST localhost
ENV NGINX_PORT 443

COPY nginx.conf /etc/nginx/conf.d.templates/
COPY nginx-ssl.conf /etc/nginx/conf.d.templates/

VOLUME ["/var/lib/tomcat", "/var/lib/postgresql", "/var/log/tomcat", "/var/log/postgresql"]

EXPOSE 80 443 8080 5432

COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh && mkdir /docker-entrypoint-initdb.d

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["start"]

