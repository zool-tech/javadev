# openjdk:8-slim
# Git
# wget
# nano
# maven

FROM debian:stretch-slim

MAINTAINER zooltech (zooltech@qq.com)

# Default to UTF-8 file.encoding
ENV LANG C.UTF-8
ENV JAVA_HOME /docker-java-home
ENV JAVA_VERSION 8u181
ENV JAVA_DEBIAN_VERSION 8u181-b13-2~deb9u1

# see https://bugs.debian.org/775775
# and https://github.com/docker-library/java/issues/19#issuecomment-70546872
ENV CA_CERTIFICATES_JAVA_VERSION 20170531+nmu1

RUN set -ex; \
# deal with slim variants not having man page directories (which causes "update-alternatives" to fail)
	if [ ! -d /usr/share/man/man1 ]; then \
		mkdir -p /usr/share/man/man1; \
	fi; \
	apt-get update \
	&& apt-get install -y --no-install-recommends bzip2 unzip xz-utils wget git nano openjdk-8-jdk-headless="$JAVA_DEBIAN_VERSION" ca-certificates-java="$CA_CERTIFICATES_JAVA_VERSION"; \
	rm -rf /var/lib/apt/lists/*; \
# add a simple script that can auto-detect the appropriate JAVA_HOME value
# based on whether the JDK or only the JRE is installed
	{ \
		echo '#!/bin/sh'; \
		echo 'set -e'; \
		echo; \
		echo 'dirname "$(dirname "$(readlink -f "$(which javac || which java)")")"'; \
	} > /usr/local/bin/docker-java-home \
	&& chmod +x /usr/local/bin/docker-java-home; \
	ln -svT "/usr/lib/jvm/java-8-openjdk-$(dpkg --print-architecture)" /docker-java-home; \
# verify that "docker-java-home" returns what we expect
	[ "$(readlink -f "$JAVA_HOME")" = "$(docker-java-home)" ]; \
# update-alternatives so that future installs of other OpenJDK versions don't change /usr/bin/java
	update-alternatives --get-selections | awk -v home="$(readlink -f "$JAVA_HOME")" 'index($3, home) == 1 { $2 = "manual"; print | "update-alternatives --set-selections" }'; \
# ... and verify that it actually worked for one of the alternatives we care about
	update-alternatives --query java | grep -q 'Status: manual'

ARG MAVEN_VERSION=3.6.0
ARG MAVEN_BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

# see CA_CERTIFICATES_JAVA_VERSION notes above
RUN echo "alias ls='ls --color=auto'\nalias ll='ls -lA'" > /root/.bashrc; \
  /var/lib/dpkg/info/ca-certificates-java.postinst configure; \
  wget ${MAVEN_BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz -P /tmp \
  && mkdir -p /usr/share/maven \
  && tar zxf /tmp/apache-maven-${MAVEN_VERSION}-bin.tar.gz -C /usr/share/maven --strip-components=1\
  && rm -f /tmp/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
