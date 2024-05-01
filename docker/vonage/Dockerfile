# Vonage Dockerfile. Builds dependencies from GitHub and then builds this repo
# Freeswitch is installed to /usr/local/freeswitch
# TLS certificates are symlinked from /usr/local/freeswitch/conf/tls
#
# To build for AMD64 arch, use: DOCKER_BUILDKIT=1 docker buildx build --platform 'linux/amd64' -f docker/vonage/Dockerfile -t <repo>:<tag> .

FROM debian:bookworm-slim
MAINTAINER Phil Lavin <phil.lavin@vonage.com>

RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get -yq install \
# build
    build-essential cmake automake autoconf 'libtool-bin|libtool' pkg-config lsb-release \
# general
    libssl-dev zlib1g-dev libdb-dev unixodbc-dev libncurses5-dev libexpat1-dev libgdbm-dev bison libtpl-dev libtiff5-dev uuid-dev \
# core
    libpcre3-dev libedit-dev libsqlite3-dev libcurl4-openssl-dev nasm \
# core codecs
    libogg-dev libspeex-dev libspeexdsp-dev \
# mod_enum
    libldns-dev \
# mod_python3
    python3-dev \
# mod_av
    libavformat-dev libswscale-dev libswresample-dev \
# mod_lua
    liblua5.2-dev \
# mod_opus
    libopus-dev \
# mod_pgsql
    libpq-dev \
# mod_sndfile
    libsndfile1-dev libflac-dev libogg-dev libvorbis-dev \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -yq install git \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN git clone --branch v1.8.3 https://github.com/signalwire/libks /usr/src/libs/libks
RUN cd /usr/src/libs/libks && cmake . -DCMAKE_INSTALL_PREFIX=/usr -DWITH_LIBBACKTRACE=1 && make && make install

RUN git clone --depth 1 --branch v1.13.17 https://github.com/freeswitch/sofia-sip /usr/src/libs/sofia-sip
RUN cd /usr/src/libs/sofia-sip && ./bootstrap.sh && ./configure CFLAGS="-g -ggdb" --with-pic --with-glib=no --without-doxygen --disable-stun --prefix=/usr && make -j`nproc --all` && make install

RUN git clone https://github.com/freeswitch/spandsp /usr/src/libs/spandsp
RUN cd /usr/src/libs/spandsp && git checkout 728b60ab && cd -
RUN cd /usr/src/libs/spandsp && ./bootstrap.sh && ./configure CFLAGS="-g -ggdb" --with-pic --prefix=/usr && make -j`nproc --all` && make install

RUN git clone --depth 1 --branch v1.3.3 https://github.com/signalwire/signalwire-c /usr/src/libs/signalwire-c
RUN cd /usr/src/libs/signalwire-c && PKG_CONFIG_PATH=/usr/lib/pkgconfig cmake . -DCMAKE_INSTALL_PREFIX=/usr && make install

ADD . / /usr/src/freeswitch
RUN cd /usr/src/freeswitch && git config pull.rebase true
RUN cd /usr/src/freeswitch && ./bootstrap.sh -j
RUN cd /usr/src/freeswitch && ./configure
RUN cd /usr/src/freeswitch && make -j`nproc` && make install

RUN ln -s /usr/local/freeswitch/conf/tls /usr/local/freeswitch/certs

CMD ["/usr/local/freeswitch/bin/freeswitch", "-nonat"]
