FROM alpine:3.6

LABEL maintainer "https://github.com/blacktop"

RUN apk add --no-cache openjdk8-jre tini su-exec

ENV VERSION 5.6.1

ENV DOWNLOAD_URL "https://artifacts.elastic.co/downloads/elasticsearch"
ENV ES_TARBAL "${DOWNLOAD_URL}/elasticsearch-${VERSION}.tar.gz"
ENV ES_TARBALL_ASC "${DOWNLOAD_URL}/elasticsearch-${VERSION}.tar.gz.asc"
ENV SHA1_URL "${DOWNLOAD_URL}/elasticsearch-${VERSION}.tar.gz.sha1"
ENV ES_TARBALL_SHA1 "31168d3e6383dd5aef03b150fc718c789b6ad1dd"
ENV GPG_KEY "46095ACC8548582C1A2699A9D27D666CD88E42B4"

RUN apk add --no-cache bash
RUN apk add --no-cache -t .build-deps wget ca-certificates gnupg openssl \
  && cd /tmp \
  && echo "===> Install Elasticsearch..." \
  && EXPECTED_SHA=$(wget -O - ${SHA1_URL}) \
  && wget -O elasticsearch.tar.gz "$ES_TARBAL"; \
	if [ "$ES_TARBALL_SHA1" ]; then \
		echo "$ES_TARBALL_SHA1 *elasticsearch.tar.gz" | sha1sum -c -; \
	fi; \
	if [ "$ES_TARBALL_ASC" ]; then \
		wget -O elasticsearch.tar.gz.asc "$ES_TARBALL_ASC"; \
		export GNUPGHOME="$(mktemp -d)"; \
		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY"; \
		gpg --batch --verify elasticsearch.tar.gz.asc elasticsearch.tar.gz; \
		rm -r "$GNUPGHOME" elasticsearch.tar.gz.asc; \
	fi; \
  tar -xf elasticsearch.tar.gz \
  && ls -lah \
  && mv elasticsearch-$VERSION /usr/share/elasticsearch \
  && adduser -DH -s /sbin/nologin elasticsearch \
  && echo "===> Creating Elasticsearch Paths..." \
  && for path in \
  	/usr/share/elasticsearch/data \
  	/usr/share/elasticsearch/logs \
  	/usr/share/elasticsearch/config \
  	/usr/share/elasticsearch/config/scripts \
  	/usr/share/elasticsearch/plugins \
  ; do \
  mkdir -p "$path"; \
  chown -R elasticsearch:elasticsearch "$path"; \
  done \
  && rm -rf /tmp/* \
  && apk del --purge .build-deps

ENV PATH /usr/share/elasticsearch/bin:$PATH

WORKDIR /usr/share/elasticsearch

# Install geoip/user-agent plugin
RUN elasticsearch-plugin install -b ingest-geoip
RUN elasticsearch-plugin install -b ingest-user-agent

COPY config/elastic /usr/share/elasticsearch/config
COPY config/logrotate /etc/logrotate.d/elasticsearch
COPY elastic-entrypoint.sh /
COPY docker-healthcheck /usr/local/bin/

VOLUME ["/usr/share/elasticsearch/data"]

EXPOSE 9200 9300
ENTRYPOINT ["/elastic-entrypoint.sh"]
CMD ["elasticsearch"]

# HEALTHCHECK CMD ["docker-healthcheck"]
