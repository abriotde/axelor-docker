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
  ./gradlew --no-daemon -x test npm-build build && 

FROM ceeedev/aio-base
LABEL maintainer="Chinthaka Deshapriya <axelor@ceee.dev>"

COPY --from=builder /app/src/open-suite-webapp/build/libs/open-suite-webapp*.war $CATALINA_BASE/webapps/ROOT.war

CMD ["start"]
