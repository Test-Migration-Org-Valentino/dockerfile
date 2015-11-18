FROM debian:jessie

ENV RUBY_MAJOR 2.2
ENV RUBY_VERSION 2.2.3
ENV RUBYGEMS_VERSION 2.5.0
ENV BUNDLER_VERSION 1.10.6
ENV RAILS_VERSION 4.2.3
ENV RUBY_DOWNLOAD_SHA256 df795f2f99860745a416092a4004b016ccf77e8b82dec956b120f18bdc71edce
ENV GEM_HOME /usr/local/bundle
ENV PATH $GEM_HOME/bin:$PATH
ENV BUNDLE_APP_CONFIG $GEM_HOME
ENV DEPLOYER_GROUP deployer
ENV DEPLOYER_GID 1001
ENV DEPLOYER_USER deployer
ENV DEPLOYER_UID 1001

RUN buildDeps=' \
		bzip2 \
		curl \
		libffi-dev \
		libgdbm3 \
		libssl-dev \
		libyaml-dev \
		procps \
		zlib1g-dev \
		autoconf \
		bison \
		gcc \
		libbz2-dev \
		libgdbm-dev \
		libglib2.0-dev \
		libncurses-dev \
		libreadline-dev \
		libxml2-dev \
		libxslt-dev \
		make \
	' \
	&& serviceDep=' \
			nodejs \
			mysql-client \
			ca-certificates \
	' \
	&& set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends $buildDeps $serviceDep\
	&& mkdir -p /usr/src/ruby \
	&& curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
	&& echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
	&& tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1 \
	&& rm ruby.tar.gz \
	&& cd /usr/src/ruby \
	&& autoconf \
	&& ./configure --disable-install-doc \
	&& make -j"$(nproc)" \
	&& make install \
	&& rm -r /usr/src/ruby && \
		groupadd -r $DEPLOYER_GROUP -g $DEPLOYER_GID && \
		useradd -u $DEPLOYER_UID -r -g $DEPLOYER_GID -d /home/$DEPLOYER_NAME -s /sbin/nologin -c \"Docker user\" $DEPLOYER_NAME && \
		chown -R $DEPLOYER_GROUP:$DEPLOYER_NAME /home/$DEPLOYER_NAME

USER $DEPLOYER_NAME
RUN	echo 'install: --no-document' >> "$HOME/.gemrc" && \
		echo 'update: --no-document' >> "$HOME/.gemrc" && \
		gem update --no-document $RUBYGEMS_VERSION && \
		gem install bundler --no-document --version "$BUNDLER_VERSION" && \
		bundle config --global path "$GEM_HOME" && \
		bundle config --global bin "$GEM_HOME/bin" && \
		gem install rails --no-document --version "$RAILS_VERSION"

USER root
RUN apt-get purge -y --auto-remove $buildDeps \
	&& rm -rf /var/lib/apt/lists/*
