FROM ceeedev/aio-builder as builder

RUN mkdir -p /app/src
WORKDIR /app/src

RUN \
  set -ex && \ 
  rm -rf open-suite-webapp && \
  git clone https://github.com/axelor/open-suite-webapp.git && \
  sed -e 's|git@github.com:|https://github.com/|' -i open-suite-webapp/.gitmodules && \
  cd open-suite-webapp && \
  git checkout master && \
  git submodule init && \ 
  git submodule update && \
  git submodule foreach git checkout master && \
  git submodule foreach git pull origin master && \
  sed -e 's|^application.locale.*|application.locale = en|g' -i src/main/resources/axelor-config.properties && \
  sed -e 's|^application.mode.*|application.mode = prod|g' -i src/main/resources/axelor-config.properties && \
  ./gradlew --no-daemon -x test npm-build build 

FROM tomcat:9-jdk11-temurin-jammy

ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH

LABEL maintainer="Chinthaka Deshapriya <axelor@ceee.dev>"

COPY --from=builder /app/src/open-suite-webapp/build/libs/open-suite-webapp*.war $CATALINA_HOME/webapps/ROOT.war

EXPOSE 8080

# upstream eclipse-temurin-provided entrypoint script caused https://github.com/docker-library/tomcat/issues/77 to come back as https://github.com/docker-library/tomcat/issues/302; use "/entrypoint.sh" at your own risk
ENTRYPOINT []

CMD ["catalina.sh", "run"]
