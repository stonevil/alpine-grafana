FROM quay.io/stonevil/alpine-glibc
MAINTAINER stonevil@gmail.com

ENV GRAFANA_VERSION=${GRAFANA_VERSION:-3.1.0-1466666977beta1}
ENV GRAFANA_DOWNLOAD_URL=${GRAFANA_DOWNLOAD_URL:-https://grafanarel.s3.amazonaws.com/builds/grafana-${GRAFANA_VERSION}.linux-x64.tar.gz}

ENV GRAFANA_BASE_PATH /opt
ENV GRAFANA_PATH=${GRAFANA_BASE_PATH}/grafana
ENV PATH $GRAFANA_PATH/bin:$PATH

LABEL com.stonevil.grafana.version=${GRAFANA_VERSION}

RUN mkdir -p ${GRAFANA_BASE_PATH} && \
  ${SOFTWARE_DOWNLOAD_CMD} ${GRAFANA_DOWNLOAD_URL} | tar -zxvf - -C ${GRAFANA_BASE_PATH} && \
  mv ${GRAFANA_BASE_PATH}/grafana-${GRAFANA_VERSION} ${GRAFANA_PATH} && \
  rm -rf ${GRAFANA_PATH}/conf/*.ini && \
  rm -rf ${GRAFANA_PATH}/*.md && \
  adduser -D -H -h /data/grafana -s /bin/bash grafana && \
  chown -Rf grafana ${GRAFANA_BASE_PATH}/grafana*

ADD _rootfs /
ONBUILD ADD _rootfs_vendor /

EXPOSE 3000

VOLUME ["/data/grafana"]

ENV GRAFANA_DASHBOARDS_INIT=${GRAFANA_DASHBOARDS_INIT:-true}
ENV GRAFANA_DASHBOARDS_PATH=${GRAFANA_DASHBOARDS_PATH:-$GRAFANA_PATH/conf/dashboards}

ENV GRAFANA_URL_SUBDIRECTORY=${GRAFANA_URL_SUBDIRECTORY:-""}

ENV GRAFANA_CONFIG_PATH=${GRAFANA_CONFIG_PATH:-$GRAFANA_PATH/conf/defaults.ini

ENV GRAFANA_ADMIN=${GRAFANA_ADMIN:-admin}
ENV GRAFANA_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD:-admin}

CMD mkdir -p /data/grafana/plugins /data/grafana/logs; chown -fR grafana:grafana /data/grafana && \
  (sed -i -e 's|_GRAFANAADMIN_|'"${GRAFANA_ADMIN}"'|g' -e 's|_GRAFANAADMINPASSWORD_|'"${GRAFANA_ADMIN_PASSWORD}"'|g' -e 's|_GRAFANAURLSUBDIRECTORY_|'"${GRAFANA_URL_SUBDIRECTORY}"'|g' ${GRAFANA_CONFIG_PATH}) && \
  (if ${GRAFANA_DASHBOARDS_INIT}; then (until [ `${SOFTWARE_DOWNLOAD_CMD} -sk -w "%{http_code}\\n" "127.0.0.1:3000" -o /dev/null` -eq 200 ]; do echo "Waiting for Grafana to init dashboards" && sleep 5; done) && \
  (if [ -d "${GRAFANA_DASHBOARDS_PATH}" ]; then echo "Initilasing dashboards"; \
  (for item in $(ls ${GRAFANA_DASHBOARDS_PATH}/*-datasource.json); do (${SOFTWARE_DOWNLOAD_CMD} -sk -u ${GRAFANA_ADMIN}:${GRAFANA_ADMIN_PASSWORD} -H 'content-type: application/json' http://127.0.0.1:3000/api/datasources -d@$item; sleep 5) done) && \
  (for item in $(ls ${GRAFANA_DASHBOARDS_PATH}/*-dashboard.json); do (${SOFTWARE_DOWNLOAD_CMD} -sk -u ${GRAFANA_ADMIN}:${GRAFANA_ADMIN_PASSWORD} -H 'content-type: application/json' http://127.0.0.1:3000/api/dashboards/db -d@$item; sleep 5) done) && \
  (sed -i -e 's|_GRAFANADASHBOARDSPATH_|'"${GRAFANA_DASHBOARDS_PATH}"'|g' ${GRAFANA_CONFIG_PATH}); else echo "Dashboards folder is absent. Dashboards will be not initialised automatically"; fi); else echo "Dashboards initialisation disabled"; fi) & \
  cd ${GRAFANA_PATH} && pwd && gosu grafana ./bin/grafana-server -config ${GRAFANA_CONFIG_PATH}
