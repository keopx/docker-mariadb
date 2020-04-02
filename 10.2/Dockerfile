FROM debian:stretch

MAINTAINER keopx <keopx@keopx.net>

ENV DEBIAN_FRONTEND noninteractive

ENV MARIADB_MAJOR 10.2

# Set repositories
RUN \
  echo "deb http://ftp.de.debian.org/debian/ stretch main non-free contrib" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.de.debian.org/debian/ stretch main non-free contrib" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
  # Update repositories cache and distribution
  apt-get -qq update && apt-get -qqy upgrade

RUN \
  apt-get -yqq install apt-transport-https lsb-release ca-certificates gnupg2 openssl wget dirmngr software-properties-common && rm -rf /var/lib/apt/lists/*

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

# grab gosu for easy step-down from root
# https://github.com/tianon/gosu/releases
ENV GOSU_VERSION 1.10

RUN set -ex; \
	fetchDeps='ca-certificates wget bzip2 gnupg dirmngr'; \
	apt-get -qq update && apt-get install -y --no-install-recommends $fetchDeps; \
	update-ca-certificates; \
	dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
	wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch"; \
	wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
	export GNUPGHOME="$(mktemp -d)" && \
	for server in $(shuf -e ha.pool.sks-keyservers.net \
   hkp://p80.pool.sks-keyservers.net:80 \
   keyserver.ubuntu.com \
   hkp://keyserver.ubuntu.com:80 \
   pgp.mit.edu) ; do \
   gpg --no-tty --keyserver "$server" --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 && break || : ; \
	done && \
	gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
	rm -fr "$GNUPGHOME" /usr/local/bin/gosu.asc; \
	chmod +x /usr/local/bin/gosu; \
	gosu nobody true
# 	&& gosu nobody true \
# 	&& gosu nobody true \
# 	&& apt-get purge -y --auto-remove ca-certificates wget

RUN mkdir /docker-entrypoint-initdb.d

# install "apt-transport-https" for Percona's repo (switched to https-only)
# install "pwgen" for randomizing passwords
# install "tzdata" for /usr/share/zoneinfo/
RUN apt-get update && apt-get install -y --no-install-recommends \
		apt-transport-https ca-certificates \
		pwgen \
		tzdata \
&& rm -rf /var/lib/apt/lists/*

# MariaDB 10.2 repository list - created 2018-12-01 12:06 UTC
RUN \
   apt-key adv --no-tty --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8  || \
   apt-key adv --recv-keys --keyserver pgp.mit.edu 0xF1656F24C74CD1D8  || \
   apt-key adv --recv-keys --keyserver keyserver.pgp.com 0xF1656F24C74CD1D8  || \
   apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8

RUN \
   add-apt-repository "deb [arch=amd64] http://mirror.netinch.com/pub/mariadb/repo/${MARIADB_MAJOR}/debian stretch main" && \
   echo "deb http://mirror.netinch.com/pub/mariadb/repo/${MARIADB_MAJOR}/debian stretch main" > /etc/apt/sources.list.d/mariadb.list && \
   echo "deb-src http://mirror.netinch.com/pub/mariadb/repo/${MARIADB_MAJOR}/debian stretch main" > /etc/apt/sources.list.d/mariadb.list

RUN \
   apt-get update && \
   apt-get -yqq install mariadb-server

RUN set -ex; \	
	rm -rf /var/lib/apt/lists/*; \
# comment out any "user" entires in the MySQL config ("docker-entrypoint.sh" or "--user" will handle user switching)
	sed -ri 's/^user\s/#&/' /etc/mysql/my.cnf /etc/mysql/conf.d/*; \
# purge and re-create /var/lib/mysql with appropriate ownership
	rm -rf /var/lib/mysql; \
	mkdir -p /var/lib/mysql /var/run/mysqld; \
	chown -R mysql:mysql /var/lib/mysql /var/run/mysqld; \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
	chmod 777 /var/run/mysqld; \
# comment out a few problematic configuration values
	find /etc/mysql/ -name '*.cnf' -print0 \
		| xargs -0 grep -lZE '^(bind-address|log)' \
		| xargs -rt -0 sed -Ei 's/^(bind-address|log)/#&/'; \
# don't reverse lookup hostnames, they are usually another container
	echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/conf.d/docker.cnf

VOLUME /var/lib/mysql

# MariaDB start command.
COPY ./scripts/docker-entrypoint.sh /usr/local/bin/
RUN ln -s /usr/local/bin/docker-entrypoint.sh / # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306
CMD ["mysqld"]
