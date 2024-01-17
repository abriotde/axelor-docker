FROM ubuntu:jammy 
LABEL maintainer="Chinthaka <axcelor@ceee.dev>"
 
RUN set -ex \
	&& DEBIAN_FRONTEND=noninteractive apt-get update \
	&& DEBIAN_FRONTEND=noninteractive apt-get upgrade -y \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -y gnupg dirmngr apt-transport-https locales \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -y openjdk-11-jdk git-core nodejs yarn \
	&& rm -rf /var/lib/apt/lists/*

# update locale
RUN set -ex && \
	echo 'en_GB.UTF-8 UTF-8' >> /etc/locale.gen && \
	echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen
ENV LANG en_US.utf8

# jdk
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64
ENV PATH "${JAVA_HOME}/bin:${PATH}"

RUN set -ex \
	&& update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 20000 \
	&& update-alternatives --install /usr/bin/javac javac $JAVA_HOME/bin/javac 20000 \
	&& update-alternatives --install /usr/bin/jar jar $JAVA_HOME/bin/jar 20000

RUN \
  set -ex && \
  mkdir -p /app/src && cd /app/src && \
  git clone https://github.com/axelor/axelor-development-kit.git && \
  cd axelor-development-kit && \
  ./gradlew --no-daemon -x test build

RUN \
  set -ex && \
  cd /app/src && \
  git clone https://github.com/axelor/abs-webapp.git axelor-erp && \
  sed -e 's|git@github.com:|https://github.com/|' -i axelor-erp/.gitmodules && \
  cd axelor-erp && \
  git checkout master && \
  git submodule sync && \
  git submodule init && \
  git submodule update && \
  git submodule foreach git checkout master && \
  git submodule foreach git pull origin master && \
  sed -e 's|^application.theme.*|application.theme = modern|g' -i src/main/resources//axelor-config.properties && \
  ./gradlew --no-daemon -x test npm-build build && \
  rm -rf /app

