# 第一阶段：构建阶段
FROM alpine:latest AS builder

ARG HTTP_PROXY
ARG HTTPS_PROXY
ARG LIBTORRENT_VERSION=1.2.19
ARG QBITTORRENTEE_VERSION=4.6.5.10

# 设置代理
ENV http_proxy=$HTTP_PROXY
ENV https_proxy=$HTTPS_PROXY

# 安装构建所需的依赖包
RUN set -x \
 && apk upgrade \
 && apk add --no-cache \
        boost-dev \
        curl \
        cmake \
        g++ \
        make \
        openssl-dev \
        qt5-qttools-dev \
        git

# 编译 libtorrent-rasterbar
RUN set -x \
 && LIBTORRENT_RASTERBAR_URL=$(curl -sSL https://api.github.com/repos/arvidn/libtorrent/releases/tags/v${LIBTORRENT_VERSION} | grep browser_download_url  | grep libtorrent-rasterbar | head -n 1 | cut -d '"' -f 4) \
 && mkdir /tmp/libtorrent-rasterbar \
 && curl -sSL $LIBTORRENT_RASTERBAR_URL | tar xzC /tmp/libtorrent-rasterbar \
 && cd /tmp/libtorrent-rasterbar/* \
 && mkdir build \
 && cd build \
 && cmake .. \
 && make install -j $(nproc)

# 编译 qBittorrent
RUN set -x \
 && QBITTORRENTEE_URL=https://github.com/c0re100/qBittorrent-Enhanced-Edition/archive/refs/tags/release-$QBITTORRENTEE_VERSION.tar.gz \
 && mkdir /tmp/qbittorrent \
 && curl -sSL $QBITTORRENTEE_URL | tar xzC /tmp/qbittorrent \
 && cd /tmp/qbittorrent/qBittorrent-Enhanced-Edition-release-$QBITTORRENTEE_VERSION \
 && ./configure --disable-gui \
 && make -j $(nproc) \
 && make install -j $(nproc)

# 第二阶段：运行阶段
FROM alpine:latest

# 安装运行所需的依赖包
RUN set -x \
 && apk upgrade \
 && apk add --no-cache \
        boost-system \
        boost-thread \
        ca-certificates \
        dumb-init \
        openssl \
        qt5-qtbase

# 从构建阶段复制编译好的文件
COPY --from=builder /usr/local/lib /usr/local/lib
COPY --from=builder /usr/local/bin /usr/local/bin

ENV HOME=/home/qbittorrent

# 添加 qbittorrent 用户
RUN set -x \
 && adduser -S -D -u 520 -g 520 -s /sbin/nologin qbittorrent \
 && mkdir -p $HOME/.config/qBittorrent \
 && mkdir -p $HOME/qbittorrent/.local/share/qBittorrent \
 && mkdir /downloads \
 && chmod go+rw -R $HOME/ /downloads \
 && ln -s $HOME/.config/qBittorrent /config \
 && ln -s $HOME/.local/share/qBittorrent /torrents \
 && su qbittorrent -s /bin/sh -c 'qbittorrent-nox -v'

USER qbittorrent
WORKDIR $HOME

# 默认配置文件
COPY qBittorrent.conf /default/qBittorrent.conf
COPY entrypoint.sh .

VOLUME ["/config", "/torrents", "/downloads"]

EXPOSE 8181 48181

ENTRYPOINT ["dumb-init", "./entrypoint.sh"]
CMD ["qbittorrent-nox"]
