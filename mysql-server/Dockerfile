FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive

RUN sed -i 's/# deb/deb/g' /etc/apt/sources.list

# Packages install
RUN apt update \
    && apt install -y systemd systemd-sysv nano vim tzdata wget bzip2 tar mysql-client curl pip python git \
    gnupg2 libaio1 libnuma-dev libgssapi-krb5-2

# Configure timezone
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

# Install restic
RUN wget https://github.com/restic/restic/releases/download/v0.13.1/restic_0.13.1_linux_amd64.bz2 \
        -O /tmp/restic_0.13.1_linux_amd64.bz2 \
    && bzip2 -d /tmp/restic_0.13.1_linux_amd64.bz2 \
    && chmod +x /tmp/restic_0.13.1_linux_amd64 \
    && mv /tmp/restic_0.13.1_linux_amd64 /usr/local/bin

# Install mysqld
RUN useradd mysql -M \
    && mkdir -p /opt/conf/percona/5.7.34-37 /opt/data/backup/5.7.34-37 \
    /opt/data/percona/5.7.34-37 /opt/lib/percona/5.7.34-37 /opt/tmp/percona/5.7.34-37 /opt/log/percona/5.7.34-37 \
    && chown mysql:mysql /opt/conf/percona/5.7.34-37 /opt/data/backup/5.7.34-37 \
    /opt/data/percona/5.7.34-37 /opt/lib/percona/5.7.34-37 /opt/tmp/percona/5.7.34-37 /opt/log/percona/5.7.34-37 \
    && chmod 0755 /opt/conf/percona/5.7.34-37 /opt/data/backup/5.7.34-37 \
    /opt/data/percona/5.7.34-37 /opt/lib/percona/5.7.34-37 /opt/tmp/percona/5.7.34-37 /opt/log/percona/5.7.34-37

RUN wget https://downloads.percona.com/downloads/Percona-Server-5.7/Percona-Server-5.7.34-37/binary/tarball/Percona-Server-5.7.34-37-Linux.x86_64.glibc2.12-minimal.tar.gz \
        -O /tmp/Percona-Server-5.7.34-37-Linux.x86_64.glibc2.12-minimal.tar.gz \
    && tar -xzf /tmp/Percona-Server-5.7.34-37-Linux.x86_64.glibc2.12-minimal.tar.gz --strip-components 1 -C /opt/lib/percona/5.7.34-37/ \
    && rm -rf /tmp/*

COPY mysqld.cnf /opt/conf/percona/5.7.34-37/
COPY mysqld-5.7.34-37.service /lib/systemd/system/
RUN /opt/lib/percona/5.7.34-37/bin/mysqld --defaults-file=/opt/conf/percona/5.7.34-37/mysqld.cnf --initialize-insecure --user=mysql
RUN systemctl enable mysqld-5.7.34-37

# Configure mysql-client
COPY my.cnf /root/.my.cnf

VOLUME [ "/sys/fs/cgroup" ]

CMD ["/lib/systemd/systemd"]