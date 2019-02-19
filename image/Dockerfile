FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=965
ARG         hostgid=965

ENV         DEBIAN_FRONTEND noninteractive
ENV         RUN_USER calibre
ENV         RUN_GROUP calibre
ENV         RUN_PORT 8005

COPY        . /tmp

# Install packages
RUN         apt-get update && \
            apt-get install --assume-yes \
                imagemagick \
                vim \
                wget \
                xdg-utils \
                xz-utils \
                xvfb && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Create folders for calibre
RUN         mkdir /calibre && \
            mkdir /calibre/data && \
            mkdir /calibre/scripts && \
            mkdir /calibre/logs && \
            mkdir /calibre/backups && \
            chown -R $RUN_USER:$RUN_GROUP /calibre
