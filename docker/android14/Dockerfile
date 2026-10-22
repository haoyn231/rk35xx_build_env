FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Asia/Shanghai \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

RUN dpkg --add-architecture i386 \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
       bc bison build-essential ccache chrpath cpio curl cvs device-tree-compiler \
       diffstat expect fakeroot file flex g++-multilib gawk gcc gcc-multilib genext2fs \
       git git-lfs gnupg gperf graphviz \
       imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev libdrm-dev \
       libelf-dev libglib2.0-dev libgtk2.0-dev liblz4-tool libncurses5-dev \
       libreadline-dev libssl-dev libxml2 libxml2-utils libxslt1-dev lzop m4 \
       make mtd-utils openssh-client patchelf pngcrush python-is-python3 \
       python3 python3-pip qemu-user-static rsync schedtool squashfs-tools subversion \
       texinfo unzip u-boot-tools wget xsltproc yasm zip zlib1g-dev \
       libc6:i386 libncurses5 libncurses5:i386 libstdc++6:i386 zlib1g:i386 libbz2-dev libffi-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && curl -fsSL https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz -o /tmp/python2.tgz \
    && tar -xzf /tmp/python2.tgz -C /tmp \
    && cd /tmp/Python-2.7.18 \
    && ./configure --prefix=/opt/python2 --enable-unicode=ucs4 --with-ensurepip=no \
    && make -j"$(nproc)" \
    && make install \
    && ln -s /opt/python2/bin/python2.7 /usr/local/bin/python2 \
    && rm -rf /tmp/Python-2.7.18 /tmp/python2.tgz \
    && useradd --create-home --uid 1000 --user-group --shell /bin/bash builder \
    && mkdir -p /workspace /home/builder/.ccache /usr/lib/jvm \
    && ln -s /workspace/prebuilts/jdk/jdk8/linux-x86 /usr/lib/jvm/java-8-openjdk-amd64 \
    && chown -R builder:builder /workspace /home/builder

ARG USER_ID=1000
ARG GROUP_ID=1000
RUN if [ "$USER_ID" != 1000 ] || [ "$GROUP_ID" != 1000 ]; then \
      groupmod --gid "$GROUP_ID" builder \
      && usermod --uid "$USER_ID" --gid "$GROUP_ID" builder \
      && chown -R "$USER_ID:$GROUP_ID" /home/builder /workspace; \
    fi

ENV HOME=/home/builder \
    PATH=/opt/python2/bin:$PATH \
    USE_CCACHE=1 \
    CCACHE_DIR=/home/builder/.ccache

USER builder
WORKDIR /workspace
CMD ["/bin/bash"]
