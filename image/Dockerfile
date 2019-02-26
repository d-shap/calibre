FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=965
ARG         hostgid=965

ENV         DEBIAN_FRONTEND noninteractive
ENV         RUN_USER calibre
ENV         RUN_GROUP calibre
ENV         RUN_PORT 8005
ENV         CALIBRE_CONFIG_DIRECTORY /calibre/config
ENV         CALIBRE_DOWNLOAD_URL https://download.calibre-ebook.com/linux-installer.sh

COPY        . /tmp

# Install packages
RUN         apt-get update && \
            apt-get install --assume-yes \
                gcc \
                python3 \
                vim \
                wget \
                xdg-utils \
                xz-utils && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Create folders for calibre
RUN         mkdir /calibre && \
            mkdir /calibre/config && \
            mkdir /calibre/data && \
            mkdir /calibre/scripts && \
            mkdir /calibre/logs && \
            mkdir /calibre/backups && \
            chown -R $RUN_USER:$RUN_GROUP /calibre

# Configure calibre
RUN         cd /tmp && \
            wget -nv -O- $CALIBRE_DOWNLOAD_URL | sh /dev/stdin && \
            calibredb add ./sample.epub --library-path /calibre/data && \
            calibredb remove 1 --permanent --library-path /calibre/data && \
            rm ./sample.epub && \
            mv /calibre/data/metadata.db /calibre

# Configure helper scripts
RUN         mv /tmp/startup /calibre/scripts && \
            mv /tmp/shutdown /calibre/scripts && \
            mv /tmp/backup /calibre/scripts && \
            mv /tmp/restore /calibre/scripts && \
            chmod 755 /calibre/scripts/*

USER        $RUN_USER

EXPOSE      $RUN_PORT

ENTRYPOINT  ["/calibre/scripts/startup"]
