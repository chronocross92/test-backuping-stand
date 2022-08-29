FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive

RUN sed -i 's/# deb/deb/g' /etc/apt/sources.list

# Packages install
RUN apt update \
    && apt install -y systemd systemd-sysv nano vim tzdata wget tar mysql-client curl pip python

# Configure timezone
RUN apt install -y tzdata
ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata

# Cleanup
RUN apt clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* 
RUN cd /lib/systemd/system/sysinit.target.wants/ \
    && ls | grep -v systemd-tmpfiles-setup | xargs rm -f $1
RUN rm -f /lib/systemd/system/multi-user.target.wants/* \
    /etc/systemd/system/*.wants/* \
    /lib/systemd/system/local-fs.target.wants/* \
    /lib/systemd/system/sockets.target.wants/*udev* \
    /lib/systemd/system/sockets.target.wants/*initctl* \
    /lib/systemd/system/basic.target.wants/* \
    /lib/systemd/system/anaconda.target.wants/* \
    /lib/systemd/system/plymouth* \
    /lib/systemd/system/systemd-update-utmp*

# Install rest-server
RUN useradd rest-server -M \
    && mkdir -p /opt/data/rest-server \
    && chown rest-server:rest-server /opt/data/rest-server

RUN wget https://github.com/restic/rest-server/releases/download/v0.10.0/rest-server_0.10.0_linux_amd64.tar.gz \
        -O /tmp/rest-server_0.10.0_linux_amd64.tar.gz \
    && tar -xzf /tmp/rest-server_0.10.0_linux_amd64.tar.gz --strip-components 1 \
    && mv rest-server /usr/local/bin/ \
    && chmod +x /usr/local/bin/rest-server \
    && rm -rf /tmp/*

COPY rest-server.service /lib/systemd/system/

RUN systemctl enable rest-server

VOLUME [ "/sys/fs/cgroup" ]

CMD ["/lib/systemd/systemd"]
