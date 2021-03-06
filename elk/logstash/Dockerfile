FROM openjdk:8-jre-alpine

# ensure logstash user exists
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
# install plugin dependencies
RUN apk add --no-cache \
# env: can't execute 'bash': No such file or directory
		bash \
		libc6-compat \
		libzmq

# grab su-exec for easy step-down from root
RUN apk add --no-cache 'su-exec>=0.2'

# https://www.elastic.co/guide/en/logstash/5.0/installing-logstash.html#_apt
# https://artifacts.elastic.co/GPG-KEY-elasticsearch
ENV GPG_KEY 46095ACC8548582C1A2699A9D27D666CD88E42B4

ENV LOGSTASH_PATH /usr/share/logstash/bin
ENV PATH $LOGSTASH_PATH:$PATH

ENV LOGSTASH_VERSION 6.4.0
#ENV LOGSTASH_TARBALL="https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.tar.gz" \
ENV LOGSTASH_TARBALL="http://10.21.8.20/soft/logstash-6.4.0.tar.gz" \
	LOGSTASH_TARBALL_ASC="http://10.21.8.20/logstash/logstash.tar.gz.asc" \
	LOGSTASH_TARBALL_SHA1="a8bf7e7ba6c1a4a0d9867bc770cd8f92"

RUN set -ex; \
	apk add --no-cache --virtual .fetch-deps \
		ca-certificates \
		gnupg \
		openssl \
		tar \
	; \
	wget -O logstash.tar.gz "$LOGSTASH_TARBALL"; \
	\
	if [ "$LOGSTASH_TARBALL_ASC" ]; then \
		wget -O logstash.tar.gz.asc "$LOGSTASH_TARBALL_ASC"; \
		export GNUPGHOME="$(mktemp -d)"; \
		gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY"; \
		gpg --batch --verify logstash.tar.gz.asc logstash.tar.gz; \
		rm -rf "$GNUPGHOME" logstash.tar.gz.asc; \
	fi; \
	\
	dir="$(dirname "$LOGSTASH_PATH")"; \
	\
	mkdir -p "$dir"; \
	tar -xf logstash.tar.gz --strip-components=1 -C "$dir"; \
	rm logstash.tar.gz; \
	\
	apk del .fetch-deps; \
	\
	export LS_SETTINGS_DIR="$dir/config"; \
# if the "log4j2.properties" file exists (logstash 5.x), let's empty it out so we get the default: "logging only errors to the console"
	if [ -f "$LS_SETTINGS_DIR/log4j2.properties" ]; then \
		cp "$LS_SETTINGS_DIR/log4j2.properties" "$LS_SETTINGS_DIR/log4j2.properties.dist"; \
		truncate -s 0 "$LS_SETTINGS_DIR/log4j2.properties"; \
	fi; \
	\
# set up some file permissions
	for userDir in \
		"$dir/config" \
		"$dir/data" \
	; do \
		if [ -d "$userDir" ]; then \
			chown -R root:root "$userDir"; \
		fi; \
	done; \
	\
	logstash --version

COPY docker-entrypoint.sh /
###########################
COPY logstash.conf /usr/share/logstash/config
COPY jvm.options /usr/share/logstash/config
COPY log4j2.properties /usr/share/logstash/config/
#RUN sed -i "s/# log.level: info/log.level: error/g" /usr/share/logstash/config/logstash.yml
#RUN sed -i "s/# log.level: info/log.level: fatal/g" /usr/share/logstash/config/logstash.yml
RUN echo "log.level: error" > /usr/share/logstash/config/logstash.yml
RUN echo "pipeline.workers: 10" >> /usr/share/logstash/config/logstash.yml
RUN echo "pipeline.output.workers: 10" >> /usr/share/logstash/config/logstash.yml
RUN echo "pipeline.batch.size: 5000" >> /usr/share/logstash/config/logstash.yml
RUN echo "pipeline.batch.delay: 10" >> /usr/share/logstash/config/logstash.yml
RUN echo "path.data: /usr/share/logstash/data" >> /usr/share/logstash/config/logstash.yml

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["-f", "/usr/share/logstash/config/logstash.conf"]